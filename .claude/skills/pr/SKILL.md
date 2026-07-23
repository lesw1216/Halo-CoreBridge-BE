---
name: pr
description: 현재 브랜치에서 PR을 생성한다. 사용자가 PR 만들어줘, PR 생성, pull request 등을 요청할 때 사용한다.
---

## PR 생성 절차

### 1. 현재 상태 확인

```bash
git status
git branch --show-current
```

미커밋 변경사항이 있으면 커밋 후 진행할지 사용자에게 확인한다.

### 2. 보호 브랜치 확인

현재 브랜치가 `dev` 또는 `main` 이면 PR 생성을 중단하고 아래를 안내한다.

```
dev / main 브랜치에서는 직접 PR을 생성할 수 없습니다.
작업 브랜치 → dev 방향으로만 PR을 생성합니다.
```

### 3. 원격 브랜치 push 확인

원격에 브랜치가 없으면 먼저 push한다.

```bash
git ls-remote --heads origin {현재 브랜치명}
```

없으면:
```bash
git push -u origin {현재 브랜치명}
```

### 4. 커밋 내역 분석

PR 제목과 본문 작성을 위해 커밋 내역을 확인한다.

```bash
git log dev..HEAD --oneline
git diff dev...HEAD --stat
```

### 5. PR 생성

브랜치 전략에 따라 base 브랜치는 반드시 `dev` 로 설정한다.
**주의**: `dev` 머지 시 GitHub Actions가 자동 배포하므로, 배포 가능한 상태인지 확인 후 진행한다.

```bash
gh pr create \
  --base dev \
  --title "{타입}: {작업 제목}" \
  --body "{본문}"
```

**PR 본문 형식:**
```markdown
## 작업 내용
- {주요 변경사항 bullet}
- {주요 변경사항 bullet}

## 변경 파일
{git diff --stat 결과 요약}

## 체크리스트
- [ ] 빌드 확인
- [ ] 주요 로직 테스트 확인
```

### 6. 결과 확인

PR URL을 사용자에게 안내한다.
