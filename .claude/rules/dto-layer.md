# DTO 레이어 규칙

요청·응답 객체는 레이어별로 역할이 분리됩니다. 네이밍을 반드시 지켜야 합니다.

> **현황**: 기존 코드는 `{Domain}Dto` 안에 nested static class를 두는 패턴(`UserDto.Create` 등)을 사용 중이다.
> 기존 코드는 현행 유지하되, **신규 작성·리팩토링하는 코드부터 본 규칙을 적용**한다.

## 네이밍 규칙

| 접미어 | 레이어 | 설명 |
|--------|--------|------|
| `**Request` | Controller | HTTP 요청 바인딩 |
| `**Command` | Service | 데이터 변경 요청 |
| `**Query` | Service | 데이터 조회·필터 요청 |
| `**Result` | Repository | Repository 반환 객체 |
| `**Response` | Controller | API 응답 페이로드 |

## DTO 흐름

```
HTTP 요청
  └─▶ **Request       (Controller 수신)
        └─▶ **Command / **Query   (Service로 전달, Controller가 변환)
                └─▶ Repository 호출
                      └─▶ **Result       (Repository 반환, Service가 수신)
                            └─▶ **Response     (Service가 변환, Controller가 반환)
                                  └─▶ BaseResponse<**Response>  (클라이언트 전달)
```

## 계층 간 독립성 원칙

- Controller는 `**Result` 를 알아서는 안 됨
- Service는 `**Request` 를 알아서는 안 됨
- Controller ↔ Service 간 Entity 직접 전달 금지
- 각 DTO는 해당 레이어 안에서만 생성·소비

## 예시 (jobposting 도메인)

```
jobPostingCreateRequest    - Controller가 HTTP Body로 수신
jobPostingCreateCommand    - Controller가 Request를 변환해 Service에 전달
jobPostingSearchQuery      - Controller가 쿼리 파라미터를 변환해 Service에 전달
jobPostingResult           - Repository가 조회 결과를 Service에 반환
jobPostingResponse         - Service가 Result를 변환해 Controller에 반환
```

---

## 패키지 구조

DTO는 도메인 패키지 아래 `dto/` 에 두고, 접미어별 서브패키지로 분리해 같은 성질의 DTO끼리 모은다.

```
api/{domain}/
├── controller/
├── service/
├── repository/
├── model/                       ← JPA Entity 전용
│   └── {Domain}.java
└── dto/
    ├── request/                 ← *Request
    ├── command/                 ← *Command
    ├── query/                   ← *Query
    ├── result/                  ← *Result
    └── response/                ← *Response
```

- 클래스의 접미어와 서브패키지 이름은 항상 일치해야 한다 (`*Request` → `dto/request/`).
- 한 도메인에서 특정 종류의 DTO가 없으면 해당 서브패키지는 만들지 않는다.
- Entity는 절대 `dto/` 아래에 두지 않는다. `model/` 에만 둔다.
- 컨테이너 클래스(`XxxDto` 안에 inner Request/Response를 두는 형태)는 신규 코드에서 금지. 각 DTO는 개별 파일로 분리한다.
