---
name: task
description: task 파일을 생성하고 TodoWrite에 등록한 뒤 작업을 시작한다.
---

파일 네이밍과 형식은 `@rules/task-driven-development.md` 참조.

## 실행 순서

### 1. 작업명 확정

사용자 요청에서 큰 단위 작업명을 추출한다.
모호하면 확인 후 진행한다.

### 2. task 파일 생성

`docs/tasks/{작업명}.md` 를 생성한다.
세부 작업 항목은 구현에 필요한 단계를 분석해 직접 작성한다.
파일 형식은 `@rules/task-driven-development.md` 참조.

### 3. TodoWrite 등록

task 파일과 동일한 항목 목록을 TodoWrite에 등록한다.

### 4. 작업 진행 중 동기화

* 각 항목 시작·완료 시 task 파일과 TodoWrite를 동시에 업데이트한다.
* 각 항목이 완료되면 커밋을 진행한다. 커밋은 `@~/.claude/skills/commit/SKILL.md` 참조
* 상태 표기는 `@rules/task-driven-development.md` 참조.

### 5. 구현 완료 후

```
1. /test    → 테스트 코드 작성 및 실행 확인
2. /done    → 빌드 확인 · 테스트 통과 · task 파일 완료 처리 · git commit
```

## 주의사항

- task 파일 없이 구현을 시작하지 않는다
- TodoWrite와 task 파일이 불일치하면 즉시 동기화한다
- 테스트 없이 `/done` 을 실행하지 않는다
