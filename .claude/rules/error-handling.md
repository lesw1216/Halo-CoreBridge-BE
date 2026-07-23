# 예외 처리 규칙

모든 예외는 `common/` 에 정의된 공통 구조를 통해 처리한다.
Service에서 `IllegalArgumentException`, `IllegalStateException` 을 직접 던지지 않는다.

---

## 구조

```
common/
├── model/
│   ├── BaseResponseStatus.java     # 에러 코드 enum
│   └── BaseResponse.java           # 공통 응답 래퍼 (success / error)
└── exception/
    ├── BaseException.java          # 공통 런타임 예외
    └── GlobalExceptionHandler.java # @RestControllerAdvice
```

`GlobalExceptionHandler` 에서 `BaseException` 을 잡아 `BaseResponse.error(status)` 형태로 반환한다.

---

## BaseResponseStatus

모든 에러 코드는 `BaseResponseStatus` enum에 정의한다. 코드 대역은 성격별로 구분한다.

| 대역 | 성격 |
|------|------|
| 20000 | 요청 성공 |
| 2000x | 인증·검증 오류 (JWT, 권한, 입력값) |
| 30000대 | Request 오류 (토큰, 파일 업로드, 파싱) |
| 40000대 | Response 오류 (조회 실패) |
| 50000대 | Database 오류 |
| 60000대 | Server 오류 |
| 70000 이상 | 도메인별 커스텀 (70000대 채용공고·파일, 71~73000대 일정, 74000대 채용 프로세스, 75000대 면접, 76000대 평가) |

```java
@Getter
public enum BaseResponseStatus {

    // 20000 : 요청 성공
    SUCCESS(true, 20000, "요청에 성공하였습니다."),

    // 2000x : 인증·검증 오류
    FIELD_VALIDATE_ERROR(false, 20001, "입력값 예외가 발생했습니다. 올바른 값을 입력하세요."),
    INVALID_JWT(false, 20002, "유효하지 않은 JWT입니다."),
    NOT_FOUND_USER(false, 20007, "존재하지 않는 사용자입니다."),

    // 50000 : Database 오류
    DATABASE_ERROR(false, 50001, "데이터베이스 연결에 실패하였습니다."),

    // 70000 : 도메인 커스텀
    JOB_POSTING_NOT_FOUND(false, 70001, "존재하지 않는 채용공고입니다."),
    INTERVIEW_NOT_FOUND(false, 75005, "존재하지 않는 면접입니다."),
    CANNOT_CANCEL_INTERVIEW(false, 75006, "취소 할 수 없는 면접입니다.");

    private final boolean isSuccess;
    private final int code;
    private final String message;

    BaseResponseStatus(boolean isSuccess, int code, String message) {
        this.isSuccess = isSuccess;
        this.code = code;
        this.message = message;
    }
}
```

새로운 도메인 추가 시 해당 도메인 블록을 위 대역 규칙에 맞춰 추가한다. 코드 중복 여부를 반드시 확인한다.

---

## BaseException

비즈니스 로직 예외는 반드시 `BaseException` 을 사용한다. 정적 팩토리 `from()` 으로 생성한다.

```java
@Getter
public class BaseException extends RuntimeException {

    private final BaseResponseStatus status;

    public BaseException(String message, BaseResponseStatus status) {
        super(message);
        this.status = status;
    }

    public static BaseException from(BaseResponseStatus status) {
        return new BaseException(status.getMessage(), status);
    }
}
```

**사용 예시:**
```java
// Service에서 throw
JobPosting jobPosting = jobPostingRepository.findById(id)
    .orElseThrow(() -> BaseException.from(BaseResponseStatus.JOB_POSTING_NOT_FOUND));

// 상태 검증
if (!interview.canBeCancelled()) {
    throw BaseException.from(BaseResponseStatus.CANNOT_CANCEL_INTERVIEW);
}
```

---

## GlobalExceptionHandler

모든 예외는 `GlobalExceptionHandler` 에서 처리하여 `BaseResponse` 형태로 반환한다.
내부 코드를 `httpStatusCodeMapper()` 로 HTTP 상태 코드에 매핑한다.

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 비즈니스 예외
    @ExceptionHandler(BaseException.class)
    public ResponseEntity<BaseResponse<Object>> handleException(BaseException e) {
        log.error("{}", e.getMessage());
        return ResponseEntity.status(httpStatusCodeMapper(e.getStatus().getCode()))
                .body(BaseResponse.error(e.getStatus()));
    }

    // @Valid 검증 예외 — 필드별 에러를 map으로 반환
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<BaseResponse<Object>> handleValidationExceptions(MethodArgumentNotValidException e) {
        Map<String, Object> errors = new HashMap<>();
        for (FieldError error : e.getBindingResult().getFieldErrors()) {
            errors.put(error.getField(), error.getDefaultMessage());
        }
        return ResponseEntity.status(httpStatusCodeMapper(e.getStatusCode().value()))
                .body(BaseResponse.error(BaseResponseStatus.FIELD_VALIDATE_ERROR, errors));
    }

    // DB 제약 위반 (FK 등)
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<BaseResponse<?>> handleDataIntegrityViolationException(DataIntegrityViolationException e) { ... }
}
```

---

## 규칙 요약

- 새로운 도메인 추가 시 관련 `BaseResponseStatus` 를 먼저 정의한다 (코드 대역·중복 확인)
- Service에서 예외 발생 시 반드시 `BaseException.from(BaseResponseStatus.XXX)` 형태로 던진다
- `IllegalArgumentException`, `IllegalStateException` 직접 사용 금지
- Controller에서 try-catch 직접 사용 금지 → GlobalExceptionHandler에 위임
- 예외 메시지를 코드 내에 하드코딩 금지 → 반드시 BaseResponseStatus에서 관리
