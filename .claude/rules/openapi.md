# OpenAPI 명세 규칙

엔드포인트 추가·변경 시 `docs/api/openapi.yaml` 을 업데이트한다.
OpenAPI 3.0 스펙을 따른다.

> **현황**: `docs/api/openapi.yaml` 은 아직 존재하지 않는다 (springdoc Swagger UI로만 문서화됨).
> 최초 작업 시 컨트롤러의 springdoc 어노테이션을 기준으로 파일을 생성하고, 이후 변경분을 반영한다.

---

## 파일 위치

```
docs/api/openapi.yaml   ← 전체 API 명세를 하나의 파일로 관리
```

---

## 기본 구조

```yaml
openapi: 3.0.3
info:
  title: CoreBridge API
  version: 1.0.0
servers:
  - url: http://localhost:8080
paths:
  ...
components:
  schemas:
    ...
```

---

## paths 작성 규칙

```yaml
paths:
  /api/job-postings:
    get:
      summary: 채용공고 목록 조회
      tags:
        - JobPosting
      responses:
        '200':
          description: 조회 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/JobPostingListResponse'
    post:
      summary: 채용공고 등록
      tags:
        - JobPosting
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/JobPostingCreateRequest'
      responses:
        '200':
          description: 등록 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/JobPostingResponse'
        '400':
          description: 입력값 오류
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
```

- `tags` 는 도메인 단위로 묶는다 (JobPosting, Interview, Evaluation, User, Organization, Schedule, Management, Resume, Board 등)
- 경로 파라미터는 `{id}` 형식으로 표기
- 경로는 실제 컨트롤러의 `/api/...` prefix를 그대로 반영한다

---

## components/schemas 작성 규칙

스키마 이름은 DTO 클래스명과 동일하게 사용한다.

```yaml
components:
  schemas:
    JobPostingCreateRequest:
      type: object
      required:
        - title
        - departmentId
        - jobType
        - headcount
      properties:
        title:
          type: string
        departmentId:
          type: integer
          format: int64
        jobType:
          type: string
        headcount:
          type: integer
          minimum: 1
        startDate:
          type: string
          format: date
        endDate:
          type: string
          format: date

    JobPostingResponse:
      type: object
      properties:
        success:
          type: boolean
        code:
          type: integer
        message:
          type: string
        results:
          $ref: '#/components/schemas/JobPostingResult'

    ErrorResponse:
      type: object
      properties:
        success:
          type: boolean
          example: false
        code:
          type: integer
        message:
          type: string
```

- 모든 응답은 `BaseResponse<T>` 래퍼를 반영한다 (`success`, `code`, `message`, `results`)
- Enum 값은 실제 열거형 값과 동일하게 작성한다 (예: `ROLE_RECRUITER`)
- `ErrorResponse` 는 공통 스키마로 한 번만 정의하고 `$ref` 로 재사용한다

---

## 공통 규칙

| 항목 | 규칙 |
|------|------|
| 포맷 | YAML |
| 버전 | OpenAPI 3.0.3 |
| 파일 | `docs/api/openapi.yaml` 단일 파일 |
| 태그 | 도메인 단위 (JobPosting, Interview, Evaluation, User, Organization 등) |
| 스키마명 | DTO 클래스명과 동일 |
| 응답 구조 | 항상 `BaseResponse<T>` 래퍼 반영 (`success` / `code` / `message` / `results`) |
| 에러 응답 | 400, 404 등 예상 가능한 오류는 반드시 명시 |
| 에러 코드 | HTTP 상태와 별개로 `BaseResponseStatus` 의 내부 코드 체계를 따름 |
