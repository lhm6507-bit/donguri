# 🌰 동구리 우편 (Donguri Postal Service)

> 소중한 마음을 엽서에 담아, 원하는 날짜에 전달하는 예약 이메일 엽서 서비스

---

## 소개

**동구리 우편**은 일본 도토리(どんぐり)를 콘셉트로 한 감성 이메일 엽서 서비스입니다.  
사용자는 엽서 템플릿을 선택하고, 내용을 작성한 뒤 발송 날짜를 예약합니다.  
예약된 날짜가 되면 수신자에게 이메일이 자동 발송되고, 수신자는 링크를 통해 BGM과 함께 엽서를 확인할 수 있습니다.

---

## 주요 기능

| 기능 | 설명 |
|---|---|
| **엽서 예약 발송** | 날짜/시간을 지정해 이메일 엽서를 예약, Quartz 스케줄러가 자동 발송 |
| **엽서 보관함** | 내가 보낸 엽서 목록 조회 및 내용 확인 |
| **템플릿 선택** | 관리자가 등록한 BASE 템플릿 + QR 스캔으로 해금하는 ADDED 템플릿 |
| **QR 코드 해금** | QR 이미지를 업로드하면 서버에서 디코딩 후 템플릿 해금 |
| **행운의 동구리 (오미쿠지)** | 하루 1회 운세 뽑기 (BAD ~ BIG_GOOD 5단계) |
| **이메일 인증 회원가입** | 6자리 인증 코드를 이메일로 발송하여 가입 |
| **BGM 재생** | 엽서 작성 및 수신 시 배경음악 선택/재생 |
| **관리자 패널** | 템플릿 등록·수정·삭제, QR 템플릿 생성, 문의 관리 |

---

## 기술 스택

| 분류 | 기술 |
|---|---|
| **Backend** | Java 8, Servlet 4.0, JSP, JSTL |
| **Build** | Gradle (WAR), Apache Tomcat |
| **Database** | Oracle DB, JDBC (ojdbc8), P6Spy (쿼리 로깅), Apache Commons DBCP2 |
| **Storage** | AWS S3 (ap-northeast-2) |
| **Scheduler** | Quartz 2.3.2 |
| **Mail** | JavaMail (Gmail SMTP) |
| **QR** | ZXing (Google) |
| **기타** | Lombok, Logback, GSON |

---

## 시작하기

### 사전 요구사항

- JDK 8+
- Apache Tomcat 9+
- Oracle Database
- AWS S3 버킷
- Gmail 앱 비밀번호 (2단계 인증 활성화 필요)

### 1. 저장소 클론

```bash
git clone <repository-url>
cd donguri
```

### 2. 환경 변수 파일 생성

`src/main/resources/.env` 파일을 생성하고 아래 내용을 채웁니다:

```env
# Oracle DB 연결 정보
DB_URL=jdbc:oracle:thin:@//호스트:1521/서비스명
DB_USER=계정명
DB_PASSWORD=비밀번호

# Gmail SMTP 설정
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_EMAIL=발신자이메일@gmail.com
SMTP_PASSWORD=Gmail앱비밀번호

# 서비스 베이스 URL (이메일 내 링크 생성에 사용)
BASE_URL=http://localhost:8080/donguri

# AWS S3
AWS_S3_ACCESS_KEY=액세스키
AWS_S3_SECRET_KEY=시크릿키
AWS_S3_BUCKET_NAME=버킷이름

# 서버 재시작 시 미발송 메일 즉시 전송 여부 (개발 환경에서는 true 권장)
SKIP_EMAIL_SEND_ON_BOOT=true
```

### 3. 데이터베이스 초기화

`src/main/java/com/c1/donguri/sql/` 디렉터리의 SQL 파일을 순서대로 실행합니다:

```
1. create.sql  → 테이블 및 트리거 생성
2. insert.sql  → 기초 데이터 삽입 (오미쿠지 운세 등)
```

### 4. 빌드 및 배포

```bash
# WAR 파일 빌드
./gradlew war

# build/libs/donguri-1.0-SNAPSHOT.war 를 Tomcat webapps에 배포
```

**Tomcat JVM 옵션 설정 필수** (한글 깨짐 방지):
```
-Dfile.encoding=UTF-8
```

---

## 프로젝트 구조

```
src/main/java/com/c1/donguri/
├── main/           # 메인 페이지, 서버 시작/종료 리스너
├── user/           # 회원가입, 로그인, 마이페이지, 이메일 인증
├── template/       # 엽서 템플릿 조회/관리, QR 생성/해금
├── reservation/    # 엽서 예약 작성·조회·수정·삭제
├── post/           # 수신 엽서 뷰, 발송함 조회
├── omikuji/        # 오미쿠지(운세) 뽑기
├── scheduler/      # Quartz 이메일 예약 잡 관리
├── inquiry/        # 문의하기 (사용자/관리자)
└── util/           # DBManager, S3Uploader, EmailSend, QRGenerator, EnvLoader

src/main/webapp/
├── main.jsp        # 공통 레이아웃 (헤더 + content 영역)
├── home.jsp        # 메인 홈 화면
├── jsp/            # 각 기능별 JSP 뷰
├── css/            # 기능별 스타일시트
├── js/             # 클라이언트 스크립트
├── image/          # UI 에셋
└── bgm/            # 배경음악 파일
```

---

## 관리자 계정

DB의 `users` 테이블에서 `roll = 'ADMIN'`으로 설정하면 관리자 권한이 부여됩니다.  
JSP에서는 `nickname = '관리자'` 여부로 관리자 메뉴 노출을 제어합니다.

관리자 전용 메뉴:
- `template-list-admin` — 전체 템플릿 관리
- `template-create-admin` — 새 템플릿 등록 (BASE / QR 해금형)
- `inquiry-admin` — 문의 목록 조회

---

## 데이터베이스 ERD (요약)

```
users ──────────────────────── reservation
  │                                │
  │ (user_template)          email_content
  │                                │
template ←──── user_template   template

omikuji    inquiry    send_log ── reservation
```

| 테이블 | 설명 |
|---|---|
| `users` | 회원 정보, `is_deleted='Y'`로 소프트 삭제 |
| `template` | 엽서 템플릿 (`BASE` / `ADDED`) |
| `email_content` | 엽서 본문 (제목, 내용, 커버이미지, BGM) |
| `reservation` | 발송 예약 (`is_done='N'→'Y'`로 완료 처리) |
| `send_log` | 발송 성공/실패 이력 |
| `user_template` | QR 해금된 템플릿-유저 관계 |
| `omikuji` | 운세 메시지 풀 |
| `inquiry` | 문의 접수 내역 |

---

## 서버 재시작 시 동작

서버가 시작되면 `AppStartupListener`가 미완료(`is_done='N'`) 예약을 전수 조회합니다:

- **이미 지난 예약** → 즉시 이메일 발송 (`.env`의 `SKIP_EMAIL_SEND_ON_BOOT=true`이면 생략)
- **아직 안 된 예약** → Quartz 잡에 등록하여 예약 시각에 자동 발송
