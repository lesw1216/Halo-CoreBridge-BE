# 환경 변수 규칙

보안이 필요한 설정값은 코드나 yml에 하드코딩하지 않고 **환경 변수**로 주입한다.
설정 파일은 `${KEY}` 형식으로만 참조한다.

---

## 파일 구성

```
corebridge-core/src/main/resources/
├── application.yml         ← 공통 설정, ${KEY} 로 환경 변수 참조
├── application-dev.yml     ← dev 프로파일 (ddl-auto: update, show-sql: true)
├── application-prod.yml    ← prod 프로파일 (ddl-auto: none, secure cookie)
└── application-token.yml   ← JWT 토큰 설정 (application.yml에서 import)
```

`corebridge-batch/src/main/resources/` 에도 동일한 구조의 yml이 별도로 있다. 공통 설정 변경 시 두 모듈을 함께 확인한다.

프로파일은 `SPRING_PROFILES_ACTIVE` 환경 변수로 선택한다.

---

## application.yml 참조 방법

```yaml
spring:
  config:
    import:
      - "classpath:application-token.yml"
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

redis:
  host: ${REDIS_HOST}
  port: ${REDIS_PORT}

upload:
  path: ${UPLOAD_PATH}
```

---

## 로컬 실행

로컬에서는 실행 시 환경 변수를 직접 주입한다 (IDE Run Configuration 또는 shell export).

```bash
export SPRING_PROFILES_ACTIVE=dev
export DB_URL=jdbc:mariadb://localhost:3306/corebridge
export DB_USERNAME=root
export DB_PASSWORD=1234
./gradlew :corebridge-core:bootRun
```

---

## 배포 환경 반영

새 환경 변수 추가 시 배포 경로에도 함께 반영해야 한다.

| 경로 | 위치 |
|------|------|
| EC2 docker compose | 서버의 compose 환경 변수 (현행 배포 경로) |
| Kubernetes | `infra/kubernetes/` 의 configMap `corebridge-config` / secret `corebridge-secret` |

---

## 규칙 요약

- DB 접속 정보, 토큰 시크릿, 외부 API 키 등 보안이 필요한 값은 반드시 환경 변수로 주입
- `application*.yml` 에 값을 직접 작성 금지 → `${KEY}` 로만 참조
- 새 환경 변수 추가 시 `application.yml`(또는 해당 프로파일 yml)과 배포 환경(compose/k8s)을 함께 수정
- 전체 환경 변수 목록은 `.claude/CLAUDE.md` 의 "필수 환경 변수" 표 참조
- **개선 대상**: `ai.base-url`, `n8n.base-url` 이 `application.yml` 에 하드코딩되어 있다. 수정 기회가 있으면 환경 변수로 분리한다
