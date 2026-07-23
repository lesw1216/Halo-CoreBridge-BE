# 테스트 작성 규칙

## 대상 및 범위

| 레이어 | 테스트 여부 | 도구 |
| ------ | ----- | ---- |
| Service | 필수 | JUnit 5 + Mockito |
| Controller | 필수 | MockMvc (`@WebMvcTest`) |
| Repository | 제외 | SQL은 통합 테스트 별도 판단 |

---

## 파일 위치

테스트 파일은 대상 클래스와 동일한 패키지 경로에 `Test` 접미사로 생성한다.

```
corebridge-core/src/main/java/com/halo/core_bridge/api/jobposting/service/JobPostingService.java
corebridge-core/src/test/java/com/halo/core_bridge/api/jobposting/service/JobPostingServiceTest.java
```

실행 명령:

```bash
# 전체 테스트
./gradlew :corebridge-core:test

# 단일 클래스
./gradlew :corebridge-core:test --tests "com.halo.core_bridge.api.jobposting.service.JobPostingServiceTest"
```

---

## 클래스 구조

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("JobPostingService")
class JobPostingServiceTest {

    @InjectMocks
    private JobPostingService jobPostingService;

    @Mock
    private JobPostingRepository jobPostingRepository;

    @Mock
    private OrganizationRepository organizationRepository;
}
```

- `@ExtendWith(MockitoExtension.class)` 사용
- 클래스 `@DisplayName`은 테스트 대상 클래스명으로 작성

---

## 메서드 네이밍

메서드명은 **테스트 대상 상황을 즉시 파악할 수 있는 영어**로 작성한다.
`@DisplayName`에 한국어로 테스트 의도를 작성한다.

```java
// ✅ 올바른 예
@Test
@DisplayName("존재하지 않는 채용공고 ID로 조회하면 예외가 발생한다")
void findById_throwsException_whenJobPostingNotFound() { ... }

@Test
@DisplayName("유효한 요청으로 채용공고를 등록하면 저장 후 응답을 반환한다")
void create_returnsResponse_whenValidCommand() { ... }

@Test
@DisplayName("취소할 수 없는 상태의 면접을 취소하면 예외가 발생한다")
void cancel_throwsException_whenNotCancellable() { ... }

// 잘못된 예
void test1() { ... }
void 공고등록테스트() { ... }
void createJobPosting() { ... }  // 상황 정보 없음
```

메서드명 패턴: `{대상메서드}_{결과}_{조건}` (조건이 명확할 때만 조건 추가)

---

## Given / When / Then 구조

모든 테스트 메서드는 `// given`, `// when`, `// then` 주석으로 블록을 구분한다.
첫 줄은 빈 줄로 시작한다 (`@rules/coding-convention.md` 준수).

```java
@Test
@DisplayName("유효한 요청으로 채용공고를 등록하면 저장 후 응답을 반환한다")
void create_returnsResponse_whenValidCommand() {

    // given
    JobPostingCreateCommand command = JobPostingCreateCommand.builder()
        .departmentId(1L)
        .title("백엔드 개발자 채용")
        .jobType("백엔드")
        .headcount(2)
        .build();

    Department department = Department.builder()
        .id(1L)
        .build();

    given(organizationRepository.findById(1L)).willReturn(Optional.of(department));

    // when
    JobPostingResponse response = jobPostingService.create(command);

    // then
    assertThat(response).isNotNull();
    assertThat(response.getTitle()).isEqualTo("백엔드 개발자 채용");
    then(jobPostingRepository).should().save(any(JobPosting.class));
}
```

---

## 예외 검증

예외 발생 케이스는 `assertThatThrownBy`로 검증한다.

```java
@Test
@DisplayName("존재하지 않는 부서 ID로 채용공고를 등록하면 예외가 발생한다")
void create_throwsException_whenDepartmentNotFound() {

    // given
    JobPostingCreateCommand command = JobPostingCreateCommand.builder()
        .departmentId(999L)
        .title("백엔드 개발자 채용")
        .build();

    given(organizationRepository.findById(999L)).willReturn(Optional.empty());

    // when & then
    assertThatThrownBy(() -> jobPostingService.create(command))
        .isInstanceOf(BaseException.class)
        .hasMessageContaining(BaseResponseStatus.DEPARTMENT_NOT_FOUND.getMessage());
}
```

---

## Mockito 스타일

BDD 스타일을 사용한다.

```java
// BDD 스타일
given(jobPostingRepository.findById(id)).willReturn(Optional.of(jobPosting));
then(jobPostingRepository).should().save(any());
then(jobPostingRepository).should(never()).delete(any());

// classic 스타일 (사용 금지)
when(jobPostingRepository.findById(id)).thenReturn(Optional.of(jobPosting));
verify(jobPostingRepository).save(any());
```

---

## Assertion 스타일

`AssertJ`를 사용한다. JUnit의 `assertEquals` 직접 사용 금지.

```java
// AssertJ
assertThat(response.getTitle()).isEqualTo("백엔드 개발자 채용");
assertThat(response.getHeadcount()).isGreaterThan(0);
assertThat(list).hasSize(3).extracting("title").contains("백엔드 개발자 채용");

// JUnit assertions (사용 금지)
assertEquals("백엔드 개발자 채용", response.getTitle());
```

---

## 공통 규칙 요약

| 항목 | 규칙 |
| ---- | ---- |
| 테스트 프레임워크 | JUnit 5 + Mockito + AssertJ |
| 클래스 어노테이션 | `@ExtendWith(MockitoExtension.class)` |
| 메서드명 | 영어, 상황을 즉시 파악 가능한 서술형 |
| DisplayName | 한국어, 테스트 의도 명확히 서술 |
| 구조 | given / when / then 주석 필수 |
| Mock 스타일 | BDD (`given`, `then`) |
| Assertion | AssertJ (`assertThat`) |
| 예외 검증 | `assertThatThrownBy` |
