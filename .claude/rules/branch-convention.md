# 브랜치 컨벤션

## 브랜치 전략

```
dev (기본 브랜치, push 시 자동 배포)
 ├── feat/#12-{브랜치명}
 ├── fix/#15-{브랜치명}
 └── ...
```

- **작업 브랜치**: `dev` 기준으로 생성, 완료 후 `dev` 로 PR
- **dev**: 기본 브랜치이자 배포 브랜치. push 시 GitHub Actions가 Docker 이미지를 빌드해 EC2에 자동 배포. 직접 커밋 금지
- `main` 브랜치는 사용하지 않는다 (릴리스 보관용)

## 브랜치 네이밍

```
{타입}/#{이슈번호}-{브랜치명}
```

### 타입

| 타입 | 설명 |
|------|------|
| `feat` | 새로운 기능 |
| `fix` | 버그 수정 |
| `refactor` | 코드 개선 |
| `chore` | 빌드·설정·의존성 변경 |
| `docs` | 문서 작성·수정 |
| `test` | 테스트 코드 작성·수정 |

### 예시

```
feat/#12-jobposting-registration
fix/#15-interview-schedule-bug
refactor/#20-dto-layer-cleanup
chore/#8-redis-setup
docs/#3-api-spec-update
test/#25-evaluation-service-test
```

## 브랜치 생성 명령

```bash
# dev 기준으로 브랜치 생성
git checkout dev
git pull origin dev
git checkout -b feat/#12-jobposting-registration
```

## PR 방향

```
작업 브랜치 → dev   (기능 완료 후, 머지 시 자동 배포)
```
