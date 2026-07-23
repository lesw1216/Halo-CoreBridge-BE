---
name: done
description: 현재 진행 중인 task를 마무리한다.
---

빌드 확인 → 테스트 통과 → task 파일 완료 처리 → git commit 순서로 실행한다.

## 실행 순서

### 1. 빌드 확인

```bash
./gradlew build -x test
```

빌드 실패 시 중단하고 오류를 보고한다. 커밋하지 않는다.

### 2. 테스트 통과

```bash
./gradlew :corebridge-core:test
```

배치 모듈을 수정했다면 `./gradlew :corebridge-batch:test` 도 함께 실행한다.
테스트 실패 시 중단하고 실패한 테스트와 원인을 보고한다. 커밋하지 않는다.

### 3. task 파일 완료 처리 및 TodoWrite 동기화

`docs/tasks/` 에서 현재 작업 중인 task 파일을 찾아 아래를 수행한다.

- 모든 항목을 `- [x]` 로 변경
- 완료일 추가 (`> 완료일: YYYY-MM-DD`)

task 파일 수정 후 TodoWrite도 동일하게 모든 항목을 `completed` 로 업데이트한다.

완료된 task 파일 예시:

```markdown
# 채용공고 등록 API

> 생성일: 2026-07-01
> 완료일: 2026-07-01
> 브랜치: feat/#12-jobposting-registration

## Tasks

- [x] JobPosting 도메인 모델 작성
- [x] JobPostingRepository 작성
- [x] DTO 작성 (Request / Command / Result / Response)
- [x] JobPostingService 작성
- [x] JobPostingController 작성
- [x] 테스트 코드 작성
- [x] 빌드 확인
```

### 4. git commit

전역 commit 스킬(`@~/.claude/skills/commit/SKILL.md`)에 위임한다.
task 파일 변경사항도 함께 스테이징 대상에 포함한다.

## 주의사항

- 빌드 또는 테스트가 실패하면 즉시 중단한다. 사용자 확인 없이 다음 단계로 넘어가지 않는다.
- task 파일이 없으면 사용자에게 알리고 커밋만 진행할지 확인한다.
- 완료일 기록 없이 커밋하지 않는다.
