### 보고서 스타일 (엄수)

**단일 HTML 파일**: 외부 CSS/JS/폰트/이미지 의존성 없음. 모든 스타일은 `<style>` 태그 안에.

**레이아웃**:

- `max-width: 880px`, 좌우 가운데 정렬
- `padding: 48px 32px` (모바일에서 `24px 16px`)
- 본문 폰트 크기 `16px`, 줄간격 `1.7`

**폰트**: 시스템 폰트 스택만 사용

```
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Apple SD Gothic Neo",
             "Malgun Gothic", system-ui, sans-serif;
```

코드는 `ui-monospace, "SF Mono", Menlo, Consolas, monospace`

**색상** (라이트 모드 기준, 다크 모드는 `prefers-color-scheme`로 자동 전환):

- 배경: `#fafafa`
- 카드/섹션 배경: `#ffffff`
- 본문 텍스트: `#1f2328`
- 보조 텍스트: `#656d76`
- 강조/링크: `#0969da`
- 경계선: `#d0d7de`
- 성공: `#1a7f37` / 경고: `#9a6700` / 위험: `#cf222e`
- 코드 배경: `#f6f8fa`

**다크 모드**:

- 배경: `#0d1117`
- 카드: `#161b22`
- 본문: `#e6edf3`
- 보조: `#8b949e`
- 강조: `#58a6ff`
- 경계선: `#30363d`
- 코드 배경: `#161b22`

**구조**:

- 최상단: 제목(H1) + 타임스탬프 + 작업 요약 한 줄
- 섹션별 `<section>` 카드: 흰 배경, `border: 1px solid <경계선>`, `border-radius: 8px`, `padding: 24px`, `margin-bottom: 16px`
- 헤딩: H1 `28px`/`600`, H2 `20px`/`600`, H3 `16px`/`600`
- 헤딩 아래 `border-bottom` 같은 장식선 추가 금지 (카드 경계로 충분)

**변경 파일 목록**:

- `<table>`로 표현: `파일 경로 | 변경 유형 | 라인 변화 | 요약`
- 변경 유형은 색 배지: 추가 `+` 녹색, 수정 `~` 파랑, 삭제 `-` 빨강
- 배지: `padding: 2px 8px`, `border-radius: 4px`, `font-size: 12px`, `font-family: monospace`

**코드 블록**:

- `<pre><code>` 구조, `background: <코드배경>`, `padding: 12px 16px`, `border-radius: 6px`, `overflow-x: auto`
- 인라인 코드도 동일 배경, `padding: 2px 6px`, `border-radius: 4px`

**금지 사항**:

- 그라데이션 배경 금지
- 박스 그림자 금지 (border로 구분)
- 이모지 금지 (단, 상태 배지의 ✓ ✗ 같은 기호는 허용)
- 애니메이션/transition 금지
- 라운드 코너 `8px` 초과 금지
- 외부 CDN 폰트(Google Fonts 등) 로드 금지
- 인라인 style 속성 사용 금지 (모두 `<style>` 안에)

**접근성**:

- `<html lang="ko">` 명시
- 본문/배경 대비 WCAG AA 이상
- 시멘틱 태그 사용 (`<main>`, `<section>`, `<header>`, `<table>`, `<th>`)
