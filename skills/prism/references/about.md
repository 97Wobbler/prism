<!-- Rendering rule: Do not narrate routing or file-loading decisions to the user — render the content below directly, without meta commentary about which intent was classified or which file was opened. -->

# Prism

Prism은 **agent / skill 설계에 쓸 수 있는 분석 도구(instrument) 카탈로그**를 제공하는 Claude Code 플러그인입니다. 플러그인 자체는 agent나 skill을 만들지 않아요 — Claude Code의 네이티브 `/agents` 플로우가 참조할 수 있는 구조화된 instrument 파일을 공급합니다.

Prism의 instrument는 다섯 가지 클래스 중 하나에 속합니다:

- **Lens (렌즈)** — 실행 가능한 분석 절차
- **Frame (분류 프레임)** — 분류/taxonomy
- **Model (이론 모델)** — 이론적·예측적 모델
- **Stance (비판적 관점)** — 해석적 commitment
- **Heuristic (원리 / 격언)** — 한 줄 rule of thumb

각 클래스의 정확한 정의·판별 기준·파일 포맷이 궁금하면 물어봐 주세요 — 루트의 `CLASSES.md`를 참조해서 부연해 드립니다.

## 바로 시작

- **새 instrument 만들기** → `/prism CVSS` 또는 `/prism 칸반 방식`
- **기존 카탈로그 탐색** → `/prism search`
- **서브에이전트에 instrument 장착** → `/prism fetch`
- **사용법 전체** → `/prism help`
