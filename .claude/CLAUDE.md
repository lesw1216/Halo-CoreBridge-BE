# CoreBridge

## 프로젝트 개요

CoreBridge는 채용 파이프라인 관리 플랫폼(ATS) 백엔드입니다. 면접 일정 관리, 이력서·평가지 뷰어, 커스텀 채용 프로세스 체인(서류→1차→2차→과제…), 칸반 파이프라인 대시보드가 핵심 기능입니다.

- **Stack**: Java 17, Spring Boot 3.5.6, MariaDB, Redis, QueryDSL, Spring Batch
- **알림**: SSE(Server-Sent Events) + Redis pub/sub
- **파일 저장**: 로컬 파일시스템 (`UPLOAD_PATH`, `utils/FileUploadUtils`)
- **API 문서**: springdoc Swagger UI
- **인프라**: Docker, GitHub Actions → DockerHub → EC2 docker compose 배포
- **Elasticsearch**: 의존성·코드는 남아 있으나 **사용 중지 상태** (`spring.data.elasticsearch.repositories.enabled: false`)

## 모듈 구성

멀티모듈 Gradle 프로젝트 (루트 프로젝트명 `corebridge`):

| 모듈 | 패키지 루트 | 역할 |
|------|-------------|------|
| `corebridge-core` | `com.halo.core_bridge` | 메인 API 서버 (port 8080) |
| `corebridge-batch` | `org.example.corebridgebatch` | 면접 알림 메일 배치 (web 미포함) |

## 빌드 및 실행 명령

```bash
# 전체 빌드
./gradlew build

# core 모듈 테스트 전체 실행
./gradlew :corebridge-core:test

# 단일 테스트 클래스 실행
./gradlew :corebridge-core:test --tests "com.halo.core_bridge.api.users.service.UserServiceTest"

# 단일 테스트 메서드 실행
./gradlew :corebridge-core:test --tests "com.halo.core_bridge.api.users.service.UserServiceTest.테스트메서드명"

# 애플리케이션 실행 (환경변수 필요)
./gradlew :corebridge-core:bootRun
```

## 필수 환경 변수

`application.yml` / `application-token.yml`에서 참조하는 환경 변수:

| 변수명 | 설명 |
|--------|------|
| `SPRING_PROFILES_ACTIVE` | 프로파일 선택 (`dev` / `prod`) |
| `DB_URL` / `DB_USERNAME` / `DB_PASSWORD` | MariaDB 접속 정보 |
| `REDIS_HOST` / `REDIS_PORT` | Redis 접속 정보 |
| `MAIL_USERNAME` / `MAIL_PASSWORD` | Gmail SMTP |
| `UPLOAD_PATH` | 파일 업로드 로컬 경로 (PDF·이미지) |
| `PWD_RESET_REDIRECT_URL` | 비밀번호 재설정 리다이렉트 URL |
| `ACCESS_TOKEN_NAME` / `ACCESS_TOKEN_EXPIRATION` / `ACCESS_TOKEN_SECRET_KEY` | Access Token 설정 |
| `REFRESH_TOKEN_NAME` / `REFRESH_TOKEN_EXPIRATION` | Refresh Token 설정 |
| `ELASTIC_URI` / `ELASTIC_USERNAME` / `ELASTIC_PASSWORD` | Elasticsearch (현재 미사용) |

## 패키지 구조 (corebridge-core)

```
com.halo.core_bridge
├── api/                        # 도메인별 패키지
│   ├── admin/                  # 계정 관리 (관리자)
│   ├── ai/                     # 외부 AI 서버 연동 (WebClient: 요약·스킬 추출·JD 매칭·점수)
│   ├── auth/                   # 이메일 인증, 이메일·비밀번호 찾기
│   ├── board/                  # 사내 공지 게시판
│   ├── coverLetterTitle/       # 자기소개서 질문
│   ├── coverLetterDescription/ # 자기소개서 답변
│   ├── evaluation/             # 면접 평가
│   ├── image/                  # 이미지 업로드
│   ├── interview/              # 면접 (장소·예약·취소)
│   ├── jobposting/             # 채용공고 (repository/es/ 는 ES용, 현재 미사용)
│   ├── mail/                   # 메일 발송
│   ├── management/             # 지원자 파이프라인 관리 (칸반)
│   ├── myPage/                 # 마이페이지
│   ├── organization/           # 부서 관리
│   ├── pdf/                    # PDF(이력서 파일) 업로드·조회
│   ├── personal/               # 지원자 개인 정보
│   ├── resume/                 # 이력서
│   ├── schedule/
│   │   ├── jobposting/         # 공고 일정
│   │   ├── jobprocess/         # 프로세스 일정
│   │   └── notification/       # 알림 (SSE, Redis pub/sub, 배치)
│   ├── token/                  # JWT (filter/, jwt/, refresh/)
│   └── users/                  # 회원 관리
├── common/
│   ├── exception/              # BaseException, GlobalExceptionHandler
│   ├── model/                  # BaseEntity, BaseResponse, BaseResponseStatus, ColorCode
│   └── serialize/              # ColorCodeSerializer
├── config/                     # Async, Batch, Elastic, QueryDsl, Redis, Security, Swagger, Web
└── utils/                      # CookieUtil, FileUploadUtils
```

각 도메인 패키지 내부 구조: `contents/`(Swagger 예시 상수) `controller/` `service/` `repository/` `model/{dto, entity, enums}`

## 아키텍처 핵심 사항

### 인증 흐름 (쿠키 기반 JWT + Redis)
- `LoginFilter`(`/api/login`) → 로그인 성공 시 Access/Refresh Token을 **쿠키**로 발급 (`CookieUtil`)
- `JwtAuthFilter`가 쿠키에서 JWT를 읽어 `UserDto.Auth`를 principal로 `SecurityContext` 설정
- `AlreadyLoginFilter`: 로그인 상태 중복 처리
- Refresh Token은 Redis에 저장 (`api/token/refresh/`)
- 토큰 설정은 `application-token.yml`에서 관리

### 권한 구조
- `UserRoleType`: `ROLE_ADMIN`(관리자) / `ROLE_APPLICANT`(지원자) / `ROLE_RECRUITER`(채용 담당자) / `ROLE_INTERVIEWER`(면접관)
- `SecurityConfig`에서 URL·메서드별 `hasRole` / `hasAnyRole` 지정, fallback은 `permitAll()`
- 컨트롤러에서 `@AuthenticationPrincipal UserDto.Auth auth`로 현재 사용자 접근

### 알림 (SSE + Redis pub/sub)
- `SseEmitterManager`(`api/schedule/notification/infra/`)가 클라이언트별 SSE 연결 관리
- `RedisNotificationPublisher/Subscriber`가 `notification_channel` 토픽으로 서버 간 알림 중계
- `corebridge-batch` 모듈이 면접 리마인드 메일을 배치로 발송 (`NotificationBatchScheduler`)

### 응답 형식
모든 API는 `BaseResponse<T>`로 감싸서 반환 (필드: `success`, `code`, `message`, `results`):
```java
BaseResponse.success(results)          // code: 20000
BaseResponse.error(status)             // BaseResponseStatus enum 참고
BaseResponse.error(status, results)    // 에러 + 부가 데이터 (예: field error map)
```
예외는 `BaseException.from(BaseResponseStatus.XXX)`로 던지고 `GlobalExceptionHandler`가 처리한다.

### QueryDSL
- 커스텀 조회는 `{X}QueryRepository` 인터페이스 + `{X}QueryRepositoryImpl` 구현 패턴 (예: `JobPostingQueryRepositoryImpl`)
- Q클래스 생성은 `./gradlew :corebridge-core:compileJava`

### AI 연동
- `api/ai/`에서 WebClient로 외부 AI 서버 호출 (이력서 요약, 스킬 추출, JD 매칭, 점수화) 및 n8n 웹훅 연동
- base-url이 `application.yml`에 하드코딩되어 있음 (개선 대상)
