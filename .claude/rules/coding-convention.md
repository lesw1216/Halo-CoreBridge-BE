# 코딩 컨벤션

## API 응답

모든 REST 응답은 `BaseResponse<T>` 래퍼를 사용한다. (필드: `success`, `code`, `message`, `results`)

```java
return ResponseEntity.ok(BaseResponse.success(response));
```

---

## Enum

- `UserRoleType`: `ROLE_ADMIN`, `ROLE_APPLICANT`, `ROLE_RECRUITER`, `ROLE_INTERVIEWER`
- 도메인별 상태 Enum은 해당 도메인 패키지의 `model/enums/` 안에서 정의한다.

---

## JPA / QueryDSL

- DB 컬럼명은 `snake_case`, Java 필드는 `camelCase` 로 작성한다.
- 단건 조회 Repository 메서드 반환 타입은 `Optional<T>` 로 선언한다.
- 커스텀 조회는 `{X}QueryRepository` 인터페이스 + `{X}QueryRepositoryImpl` 구현 패턴을 따른다.

```java
// Repository 인터페이스
Optional<JobPosting> findById(Long id);

// Service에서 사용
JobPosting jobPosting = jobPostingRepository.findById(id)
    .orElseThrow(() -> BaseException.from(BaseResponseStatus.JOB_POSTING_NOT_FOUND));
```

---

## CORS

운영 환경의 두 origin만 허용한다.

- `http://localhost:5173` — 로컬 프론트엔드 개발 환경
- `https://www.core-bridge.co.kr` — 프로덕션 도메인

새 origin 추가가 필요하면 반드시 협의 후 `SecurityConfig.corsConfigurationSource()`에 명시한다.

---

## 예외 처리

`BaseException.from(BaseResponseStatus.XXX)` 만 사용한다. `IllegalArgumentException`, `IllegalStateException` 직접 throw 금지.
상세 규칙은 `@rules/error-handling.md` 참조.

---

## DTO

Entity를 Controller 응답 또는 Response DTO로 직접 반환하지 않는다.
DTO 레이어 분리 규칙은 `@rules/dto-layer.md`, 작성 방법은 `@rules/dto-create.md` 참조.

---

## 메서드 작성 규칙

### 명명 규칙

#### A. CRUD 동사 표준화

같은 동작은 같은 동사로 통일한다.

| 동작 | 권장 동사 | 비고 |
|------|-----------|------|
| Create / Insert (Repository) | `save`, `saveAll` | Spring Data 표준 |
| Create / Insert (Service public API) | `register*` 또는 `create*` 중 도메인 단위로 통일 | 한 도메인 안에서는 하나만 사용 |
| Read 단건 | `findBy*`, `find*ById` | `get*`은 단순 getter 외 금지 |
| Read 복수 | `findAll*`, `findAllBy*` | `list*` 금지 (Spring Data 표기와 어긋남) |
| Update | `update*`, `change*` | `modify*` 금지 |
| Delete | `delete*` | `remove*` 금지 |

#### B. Boolean 메서드 prefix

`boolean` 또는 `Boolean`을 반환하는 메서드는 반드시 `is*` / `has*` / `can*` 중 하나로 시작한다. `check*`는 boolean 반환에 사용하지 않는다.

```java
// 올바른 예
boolean isCancelled();
boolean hasAppliedJobPosting(Long userIdx, Long jobPostingIdx);
boolean canBeCancelled();

// 잘못된 예
boolean checkApply(...);      // → hasApplied
boolean scheduled(...);       // → isScheduled
```

#### C. 검증·변환 prefix

| Prefix | 의미 | 반환 |
|--------|------|------|
| `validate*` | 비즈니스 규칙 검증, 실패 시 `BaseException` throw | `void` |
| `to*` | 인스턴스 메서드 변환 (`request.toCommand()`, `entity.toResult()`) | 변환된 객체 |
| `from*` | 정적 팩토리 (`Result.from(entity)`) | 생성된 객체 |

`check*`는 사용하지 않고 `validate*`(예외 throw) 또는 `is*`(boolean 반환)로 대체한다.

#### D. Repository — Spring Data 명명 규칙 준수

Repository 메서드는 Spring Data가 인식하는 prefix를 따른다.

- `findBy{Field}`, `findAllBy{Field}`
- `existsBy{Field}`
- `countBy{Field}`
- `deleteBy{Field}`

커스텀 메서드(`@Query` 사용 등)도 위 prefix 패턴을 동일하게 따른다. 도메인-축약·오타 금지 (`jpsPaging` ❌).

#### E. 일반 스타일

- 메서드명은 동사로 시작한다 (예외: `to*` / `from*` / `is*` / `has*` / `can*`)
- camelCase를 엄수한다 (`listpaging` ❌ → `listPaging`)
- 약어는 첫 글자만 대문자 (`URL` → `Url`, `JWT` → `Jwt`, `PDF` → `Pdf`)
- 의미가 모호한 동사는 금지: `do*` / `handle*` / `process*` / `perform*` (Webhook 핸들러 같은 단순 디스패처에만 허용)
- 오타·축약은 PR 리뷰에서 즉시 지적 대상

#### F. 엔티티 상태 변경은 엔티티 내부 행위 메서드로

엔티티의 필드 변경 로직은 엔티티 내부에 명시적 비즈니스 메서드로 정의한다. Service는 setter나 필드를 직접 조작하지 않고, 엔티티가 노출하는 행위 메서드를 호출한다 (Rich Domain Model).

**원칙**
- 엔티티는 자신의 상태 전이 로직을 스스로 캡슐화한다
- Service는 엔티티를 조회한 뒤, 엔티티 메서드를 호출하는 패턴으로만 상태를 바꾼다
- 메서드명은 그 행위가 의미하는 비즈니스 동작을 그대로 드러낸다 (`setPassword` ❌ → `changePassword` ✓)

**예시**

```java
// 잘못된 예 — Service가 setter를 직접 호출
public void resetPassword(...) {

    Users user = userRepository.findByEmail(email)
            .orElseThrow(() -> BaseException.from(BaseResponseStatus.NOT_FOUND_USER));

    user.setPassword(passwordEncoder.encode(newPassword));  // setter
    userRepository.save(user);
}

// 올바른 예 — 엔티티의 행위 메서드를 호출
public void resetPassword(...) {

    Users user = userRepository.findByEmail(email)
            .orElseThrow(() -> BaseException.from(BaseResponseStatus.NOT_FOUND_USER));

    user.updatePassword(passwordEncoder.encode(newPassword));
}

// Users Entity
public void updatePassword(String encodedPassword) {

    this.password = encodedPassword;
}
```

**적용 대상**
- 엔티티 필드 변경 전반 (setter 패턴)
- 상태 전이 (면접 취소, 지원 단계 이동, 채용공고 마감 등)
- 카운터·집계 갱신 (`incrementApplyCount` 등)

**예외**
- JPA 내부용 setter가 꼭 필요하면 `protected` 또는 `private`로 한정한다. 클래스 레벨 `@Setter` 금지.
- 빌더 패턴은 생성 시점 한정으로 허용
- 제3자 라이브러리 객체(`MimeMessageHelper.setTo` 등)는 본 규칙 대상이 아니다

---

### Javadoc

모든 `public` 메서드에 Javadoc을 작성한다. `private` 메서드는 이름만으로 의도가 충분히 전달되면 생략 가능하다.

- 첫 줄: 메서드가 하는 일을 한 문장으로 작성한다 (한국어)
- `@param`: 파라미터마다 작성한다 (타입 생략, 의미 한 줄)
- `@return`: 반환값이 있으면 작성한다 (`void` 제외)
- `@throws`: 의도적으로 던지는 예외만 작성한다

```java
/**
 * 채용공고를 등록하고 생성된 공고 정보를 반환한다.
 *
 * @param command 공고 등록에 필요한 데이터
 * @return 생성된 공고 응답
 * @throws BaseException 부서가 존재하지 않을 때
 */
public JobPostingResponse create(JobPostingCreateCommand command) { ... }

/**
 * 면접을 취소하고 취소된 결과를 반환한다.
 *
 * @param id      취소할 면접 ID
 * @param command 취소 요청 정보
 * @return 취소된 면접 응답
 * @throws BaseException 면접이 없거나 취소할 수 없는 상태일 때
 */
public InterviewResponse cancel(Long id, InterviewCancelCommand command) { ... }
```

---

### 메서드 첫 줄 공백

메서드 시그니처 바로 아래 줄은 반드시 빈 줄로 시작한다.

```java
// 올바른 예
public JobPostingResponse create(JobPostingCreateCommand command) {

    Department department = departmentRepository.findById(command.getDepartmentId())
        .orElseThrow(() -> BaseException.from(BaseResponseStatus.DEPARTMENT_NOT_FOUND));

    JobPosting jobPosting = buildJobPosting(command, department);
    jobPostingRepository.save(jobPosting);

    return JobPostingResponse.from(JobPostingResult.from(jobPosting));
}

// 잘못된 예
public JobPostingResponse create(JobPostingCreateCommand command) {
    Department department = departmentRepository.findById(command.getDepartmentId())
        .orElseThrow(() -> BaseException.from(BaseResponseStatus.DEPARTMENT_NOT_FOUND));
    ...
}
```

### 문맥별 공백 구분

논리적으로 다른 작업 사이에는 빈 줄을 삽입하여 가독성을 높인다.

```java
public InterviewResponse cancel(Long id) {

    Interview interview = interviewRepository.findById(id)
        .orElseThrow(() -> BaseException.from(BaseResponseStatus.INTERVIEW_NOT_FOUND));

    validateCancellable(interview);

    interview.cancel();
    interviewRepository.save(interview);

    return InterviewResponse.from(InterviewResult.from(interview));
}
```

### 파라미터 줄바꿈

파라미터가 2개 이상이고 한 줄에 담기 어려울 때, 첫 번째 파라미터는 메서드명과 같은 줄에 두고 두 번째 파라미터부터 줄바꿈한다.
들여쓰기는 첫 번째 파라미터의 시작 열에 맞춘다.

```java
// 올바른 예 — 첫 번째는 같은 줄, 두 번째부터 줄바꿈
public List<JobPostingResponse> search(Long departmentId,
                                       String jobType,
                                       String keyword) {

    return jobPostingRepository.search(departmentId, jobType, keyword);
}

void save(@Param("userId") Long userId,
          @Param("jobPostingId") Long jobPostingId,
          @Param("processIdx") Integer processIdx);

// 파라미터가 1개이거나 한 줄에 들어오면 줄바꿈 불필요
public JobPostingResponse findById(Long id) { ... }

// 잘못된 예 — 첫 번째 파라미터부터 줄바꿈
public List<JobPostingResponse> search(
        Long departmentId,
        String jobType,
        String keyword) { ... }
```

---

### 메서드 최대 10줄

메서드 바디가 10줄을 초과하면 반드시 별도 메서드로 추출한다.
단, 빈 줄과 닫는 중괄호는 줄 수에 포함하지 않는다.

```java
// 추출 후
public JobPostingResponse create(JobPostingCreateCommand command) {

    Department department = findDepartmentOrThrow(command.getDepartmentId());
    JobPosting jobPosting = buildJobPosting(command, department);

    jobPostingRepository.save(jobPosting);

    return toResponse(jobPosting);
}

private Department findDepartmentOrThrow(Long departmentId) {

    return departmentRepository.findById(departmentId)
        .orElseThrow(() -> BaseException.from(BaseResponseStatus.DEPARTMENT_NOT_FOUND));
}

private JobPosting buildJobPosting(JobPostingCreateCommand command, Department department) {

    return JobPosting.builder()
        .department(department)
        .title(command.getTitle())
        .jobType(command.getJobType())
        .careerLevel(command.getCareerLevel())
        .build();
}
```
