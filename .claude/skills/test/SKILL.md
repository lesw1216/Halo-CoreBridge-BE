---
name: test
description: 현재 구현된 코드에 대한 테스트 코드를 작성하고 실행을 확인한다.
---

테스트 작성 규칙은 `@rules/testing.md` 참조.

## 실행 순서

### 1. 테스트 대상 파악

현재 구현된 Service, Controller 클래스를 확인한다.
이미 테스트 파일이 존재하면 누락된 케이스를 추가한다.

### 2. 테스트 파일 생성

대상 클래스와 동일한 패키지 경로에 `Test` 접미사로 생성한다.

```
대상: corebridge-core/src/main/java/com/halo/core_bridge/api/jobposting/service/JobPostingService.java
생성: corebridge-core/src/test/java/com/halo/core_bridge/api/jobposting/service/JobPostingServiceTest.java
```

### 3. 테스트 케이스 작성

각 public 메서드에 대해 아래 케이스를 최소한 작성한다.

- 정상 케이스 (happy path)
- 존재하지 않는 ID / 데이터 없음 → BaseException 발생
- 유효하지 않은 상태 전이 → BaseException 발생 (해당되는 경우)

작성 규칙은 `@rules/testing.md` 참조.

### 4. 테스트 실행

```bash
./gradlew :corebridge-core:test
```

배치 모듈을 수정했다면 `./gradlew :corebridge-batch:test` 도 함께 실행한다.
실패한 테스트가 있으면 원인을 파악하고 수정한 뒤 재실행한다.
모든 테스트가 통과해야 완료로 처리한다.

### 5. task 파일 동기화

`docs/tasks/` 의 현재 task 파일에서 `테스트 코드 작성` 항목을 `completed` 로 변경한다.
TodoWrite도 동일하게 업데이트한다.

## 주의사항

- 테스트 통과 없이 `/done` 을 실행하지 않는다
