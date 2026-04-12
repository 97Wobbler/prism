---
name: fetch
description: Prism instrument를 서브에이전트/에이전트/스킬에 전달할 수 있는 형태로 준비. instrument 이름을 받아 절대경로를 resolve하고, 서브에이전트 프롬프트에 삽입할 지시문 블록(경로 목록 + 읽기 지시 + 간단 요약)을 반환한다. /prism search로 instrument를 탐색한 뒤 이 스킬로 장착하는 2단계 워크플로우. 트리거: "/prism fetch stride owasp-top10", "instrument 로드해줘", "서브에이전트에 렌즈 장착해줘", "fetch stride owasp-top10", "이 instrument를 서브에이전트에 넘겨줘"
---

# fetch

선정된 Prism instrument를 서브에이전트 프롬프트에 삽입할 수 있는 지시문 블록으로 준비한다.

## 역할

`/prism search`가 "뭐가 있는지 탐색"하는 스킬이라면,
`fetch`는 **"선정된 것을 가져오기"** 하는 스킬이다.

- 서브에이전트가 instrument를 실제로 읽고 절차를 따를 수 있도록 **경로와
  읽기 지시를 준비**한다.
- 직접 instrument 내용을 인라인으로 넣지 않는다 — **절대경로 + 읽기 지시 +
  간단 요약**을 반환할 뿐이다.
- 일반적인 워크플로우: `/prism search`로 후보를 탐색 -> `fetch`로 선정된
  instrument를 장착.

## 입력

다음 중 하나 이상을 받는다:

- **instrument 이름 (slug) 1개 이상** — `stride`, `owasp-top10`,
  `systems-thinking` 등 `catalog.yml`에 등록된 `name` 값.
- **도메인+클래스 조합** — "security lens 전부", "education frame 전부"처럼
  도메인과 클래스를 함께 지정하면 해당하는 모든 instrument를 fetch.
- 이름이 모호하면 `catalog.yml`에서 후보를 보여주고 **1개 확인 질문**만 한다.
  확인 질문 없이 추측으로 진행하지 않는다.

## Resolve 알고리즘

### 3-layer 경로 탐색

Prism은 세 계층에 instrument를 저장한다. 우선순위 **project > global >
bundle**로 병합한다:

1. **Bundle layer (read-only)** — 플러그인에 동봉된 기본 카탈로그.
   - Catalog: `<plugin-root>/catalog.yml`
   - Library: `<plugin-root>/library/`

2. **Global layer (optional)** — 사용자 전역 instrument.
   - Catalog: `~/.claude/prism/catalog.yml`
   - Library: `~/.claude/prism/library/`
   - 파일이 없으면 이 계층을 조용히 건너뛴다.

3. **Project layer (optional)** — 현재 프로젝트 전용 instrument.
   - Catalog: `./.claude/prism/catalog.yml` (CWD 기준)
   - Library: `./.claude/prism/library/`
   - 파일이 없으면 이 계층을 조용히 건너뛴다.

### 병합 및 경로 확정

- 세 계층의 `catalog.yml`을 읽어 요청된 instrument의 `path` 필드를 확보한다.
- `name` 충돌 시 project > global > bundle 순으로 상위 계층이 우선한다.
  override가 발생하면 출력에 한 줄 주석으로 표기한다.
- `path`를 해당 계층의 루트 기준으로 **절대경로**로 변환한다.
- 절대경로로 변환된 파일의 **존재 여부를 확인**한다 (Glob 또는 Read).
  존재하지 않으면 경고 메시지를 출력하고 해당 항목을 스킵한다.
- 각 instrument에 대해 frontmatter의 `one_liner`과 `class`를 간단 요약으로
  추출한다.

## 출력 형태

서브에이전트 프롬프트에 **복사해서 바로 삽입할 수 있는 텍스트 블록**을
반환한다. 형식:

```
## Prism Instruments — 서브에이전트 지시문

아래 instrument 파일들을 반드시 Read 도구로 읽고, 각 파일의 핵심 절차
섹션을 분석 워크플로우로 따르세요.

| # | name | class | 요약 | 경로 |
|---|------|-------|------|------|
| 1 | stride | lens | 위협 모델링에서 STRIDE 카테고리로 위협 식별 | /full/path/to/stride.md |
| 2 | owasp-top10 | lens | OWASP Top 10 체크리스트 실행 | /full/path/to/owasp-top10.md |

### 클래스별 핵심 섹션 가이드
- **lens**: "Analytical Procedure" 섹션의 단계를 순서대로 실행
- **frame**: "Categories" + "Classification Procedure" 적용
- **model**: "Core Variables" + "Application Procedure" 적용
- **stance**: "Guiding Questions"로 분석 관점 설정
- **heuristic**: "The Rule" + "When It Applies"를 체크포인트로 활용
```

override가 발생했다면 테이블 아래에 주석을 추가한다:

```
notes:
  - 'stride' in project layer overrides bundle entry (layer=project used)
```

## 운영 규칙

- **READ-ONLY**: 파일 생성/수정 절대 금지. 카탈로그와 라이브러리를 읽기만
  한다.
- `catalog.yml` 최신 상태를 **매 호출마다 새로 읽는다** (캐시하지 않음).
- instrument **10개 초과** 요청 시 경고 메시지를 먼저 출력하고, 사용자가
  확인하면 계속 진행한다.
- 전달 방식은 **경로 기반**: 서브에이전트가 직접 Read 도구로 파일을 읽도록
  유도한다. instrument 본문을 인라인으로 붙여넣지 않는다.

## NOT this skill

- **카탈로그 탐색/검색** — `/prism search` 스킬의 영역이다. 어떤
  instrument가 있는지 알아보려면 search를 먼저 사용한다.
- **instrument 생성** — `/prism` 스킬의 영역이다. 새 lens/frame/model/
  stance/heuristic를 만들려면 `/prism <framework>` 형태로 호출한다.
- **에이전트/스킬 직접 생성** — Claude Code 네이티브 `/agents` 플로우를
  사용한다. Prism은 카탈로그와 instrument를 공급할 뿐, 에이전트 자체를
  생성하지 않는다.
