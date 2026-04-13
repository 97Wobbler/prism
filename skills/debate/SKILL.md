---
name: debate
description: 여러 에이전트 페르소나가 문서/제안/문제를 독립적으로 분석하고 라운드를 거쳐 합의를 도출하는 오케스트레이션 스킬. 3개 모드 지원 — review(다각도 분석→판정), ideate(발산적 아이디어 탐색), solve(문제 분석→해결책 수렴). 모드를 명시하지 않으면 맥락에서 자동 추론. Prism instrument(stance/lens 등)를 토론 참여자의 전문성 소스로 활용 가능. 트리거: "다각도 분석해줘", "에이전트들끼리 토론해줘", "debate", "consensus", "여러 관점에서 검토해줘", "해결책 찾아줘", "아이디어 발산해줘", "브레인스토밍해줘", "리뷰해줘 여러 관점으로"
---

# debate

여러 페르소나 에이전트가 주어진 대상을 독립적으로 분석하고, 오케스트레이터가
라운드를 통제하며 합의·수렴·발산을 이끌어내는 오케스트레이션 엔진.

---

## 모드

| 모드 | 목적 | 종료 조건 | 대표 시나리오 |
|---|---|---|---|
| `review` | 문서/제안을 다각도로 분석해 판정 | 합의 달성 또는 MAX_ROUNDS 소진 | 코드 리뷰, 설계 검토, PRD 검토 |
| `ideate` | 아이디어를 최대한 발산·확장 | MAX_ROUNDS 소진 (수렴 판정 없음) | 브레인스토밍, 아이디에이션, 탐색적 분석 |
| `solve` | 문제에 대한 해결책을 제안하고 수렴 | 수렴 달성 또는 MAX_ROUNDS 소진 | 버그 해결, 장애 대응, 전략 수립 |

### 모드 결정 규칙

1. **명시적 플래그** — 사용자가 `--review`, `--ideate`, `--solve`를 지정하면 해당 모드로 즉시 진입
2. **맥락 추론** — 플래그가 없으면 아래 신호로 판단:
   - 분석/검토/리뷰/판정 → `review`
   - 아이디어/발산/브레인스토밍/탐색 → `ideate`
   - 해결/고치기/대안/전략 → `solve`
3. **불확실하면 `review`** — 가장 범용적인 기본값
4. **모드 전환** — `review` 완료 후 미해소 쟁점이 남고 사용자가 "해결책도 찾아줘"라고 하면 `solve`로 자연스럽게 전환 가능. 단, **자동 전환은 금지** — 반드시 사용자 확인 후 진행

---

## 전제 조건

- **분석 대상**: 문서, 텍스트, 코드, 제안, 문제 설명 등
- **참여 에이전트**: 아래 두 소스 중 하나 이상으로 구성
  - `.claude/agents/` 내 에이전트 파일 (프로젝트 또는 사용자 루트)
  - Prism instrument (특히 `stance`, `lens`) — `/prism fetch`로 로드한 instrument를 페르소나의 전문성 소스로 활용
- **에이전트가 명시되지 않으면 사용자에게 확인 후 진행** (자동 선택 금지)

### Prism 연동

사용자가 "prism instrument로 토론해줘" 또는 특정 instrument 이름을 언급하면:
1. `/prism fetch`로 해당 instrument의 경로와 요약을 확보
2. 각 subagent 프롬프트에 instrument 읽기 지시를 삽입
3. subagent는 instrument의 절차/관점을 자신의 분석 프레임으로 사용

연동이 없으면 순수 에이전트 페르소나 기반으로 동작한다. Prism은 선택적 강화이지 필수 의존성이 아니다.

---

## 실행 흐름

### STEP 0 — 세션 초기화

오케스트레이터(메인 세션)가 수행:

1. 분석 대상을 `[TARGET]` 블록으로 준비 (문서, 문제 설명 등)
2. 모드 결정 (위 규칙 적용)
3. 참여 에이전트 목록 확인 및 역할 레이블 부여
4. 설정값 확정:
   - `MAX_ROUNDS`: 기본 4 (사용자 지정 가능)
   - `CONSENSUS_THRESHOLD`: 기본 0.75 (review/solve 모드에서 사용)
5. 상태 초기화: `round = 1`, `issues = []`, `solution_pool = []`

---

### STEP 1 — 라운드 실행 (반복)

각 라운드마다 **모든 참여 에이전트를 Agent tool로 병렬 spawn**.
모드에 따라 다른 프롬프트 템플릿을 사용한다.

#### 프롬프트 공통 헤더

```
당신은 [AGENT_NAME] 페르소나입니다.
역할 설명: [AGENT_DESCRIPTION]
[PRISM_INSTRUMENT_DIRECTIVE — 있을 경우만]

## 분석 대상
[TARGET]

## 현재 라운드
Round [N] / [MAX_ROUNDS]

## 이전 라운드 요약 (Round 1이면 생략)
[PREVIOUS_ROUND_SUMMARY]
```

#### 모드별 임무 및 출력 포맷

---

### MODE: review

**임무 블록:**
```
## 당신의 임무
1. 대상을 당신의 페르소나 관점에서 분석하라.
2. 이전 라운드가 있다면, 다른 에이전트의 주장을 검토하고 동의/반박을 명시하라.
3. 아래 JSON 포맷을 정확히 따를 것. JSON 외 텍스트 출력 금지.
```

**출력 JSON:**
```json
{
  "agent": "[AGENT_NAME]",
  "round": "[N]",
  "analysis": {
    "key_findings": ["핵심 발견사항 (한 문장씩)"],
    "concerns": ["우려사항 또는 문제점"],
    "strengths": ["강점 또는 긍정적 측면"]
  },
  "stance_on_pending_issues": [
    {
      "issue": "잔류 쟁점 텍스트",
      "position": "agree | disagree | neutral",
      "rationale": "입장 근거 (2-3문장)"
    }
  ],
  "new_issues": [
    {
      "issue": "새롭게 제기하는 쟁점",
      "severity": "critical | major | minor",
      "rationale": "왜 중요한지"
    }
  ],
  "overall_recommendation": "approve | approve_with_conditions | reject",
  "recommendation_rationale": "최종 권고 이유 (3-5문장)"
}
```

---

### MODE: ideate

**임무 블록:**
```
## 당신의 임무
1. 대상을 당신의 페르소나 관점에서 분석하고, 가능한 한 다양한 아이디어를 제안하라.
2. 다른 에이전트의 아이디어에서 영감을 받아 확장·변형·결합하라. 비판보다 확장을 우선하라.
3. 기존 아이디어와 겹치지 않는 새로운 각도를 찾으려 노력하라.
4. 아래 JSON 포맷을 정확히 따를 것.
```

**출력 JSON:**
```json
{
  "agent": "[AGENT_NAME]",
  "round": "[N]",
  "perspective": "이 에이전트가 대상을 바라보는 고유 관점 (1-2문장)",
  "ideas": [
    {
      "title": "아이디어 제목",
      "description": "상세 설명 (2-3문장)",
      "novelty": "high | medium | low",
      "inspired_by": "영감을 받은 다른 에이전트의 아이디어 (없으면 null)",
      "tags": ["분류 태그"]
    }
  ],
  "combinations": [
    {
      "source_ideas": ["결합 대상 아이디어 제목들"],
      "combined_idea": "결합으로 탄생한 새 아이디어",
      "synergy": "결합의 시너지 설명"
    }
  ],
  "unexplored_angles": ["아직 다뤄지지 않은 탐색 방향 제안"]
}
```

---

### MODE: solve

**임무 블록 (Round 1):**
```
## 당신의 임무
1. 문제를 당신의 페르소나 관점에서 분석하고 해결책을 제안하라.
2. 아래 JSON 포맷을 정확히 따를 것.
```

**임무 블록 (Round 2+):**
```
## 당신의 임무
1. 다른 에이전트의 해결책을 검토하라.
2. 수용 가능 요소, 개선 필요 요소, 수용 불가 요소를 명시하라.
3. 이전 제안들을 통합·개선한 새 버전을 제시하라.
4. 이전 자기 제안 중 철회/수정할 것이 있으면 명시하라.
```

**출력 JSON:**
```json
{
  "agent": "[AGENT_NAME]",
  "round": "[N]",
  "problem_framing": "문제를 바라보는 핵심 관점 (1-2문장, Round 1만)",
  "proposed_solution": {
    "title": "해결책 제목",
    "description": "상세 설명 (3-5문장)",
    "key_steps": ["실행 단계"],
    "feasibility": "high | medium | low",
    "feasibility_rationale": "실현 가능성 근거"
  },
  "review_of_others": [
    {
      "agent": "검토 대상 에이전트",
      "elements_accepted": ["수용 가능 요소"],
      "elements_rejected": [
        { "element": "수용 불가 요소", "reason": "거부 이유" }
      ],
      "suggested_improvement": "개선 제안 (없으면 null)"
    }
  ],
  "tradeoffs": [
    {
      "tradeoff": "트레이드오프",
      "severity": "critical | major | minor",
      "mitigation": "완화 방안 (없으면 null)"
    }
  ],
  "convergence_signal": {
    "willing_to_adopt": ["통합 가능 요소"],
    "dealbreakers": ["수용 불가 요소"]
  },
  "revised_from_previous": "이전 라운드 대비 변경사항 (Round 1이면 null)"
}
```

---

### STEP 2 — 판정 (오케스트레이터)

각 라운드 종료 후 오케스트레이터가 수집된 JSON 결과를 분석한다.
**판정 로직은 모드에 따라 다르다.**

#### review 모드 — 합의 판정

**1. Recommendation 합의율 (정량)**

| overall_recommendation 분포 | 판정 |
|---|---|
| 동일 값 >= 75% | 방향 합의 |
| 동일 값 50~74% | 부분 합의 (조건부 진행) |
| 동일 값 < 50% | 합의 미달 |

**2. 잔류 쟁점 해소율 (정량)**

```
해소율 = (이전 pending_issues 중 모든 에이전트가 agree한 건수) / (전체 pending_issues 수)
```
- >= 0.8 → 충분
- < 0.8 → 다음 라운드 필요

**3. Critical 쟁점 잔존 여부 (우선 적용)**
- `severity: critical`인 `new_issues`가 하나라도 미해소 → 합의 불가 (차단 조건)

**종료 조건 (AND):** 합의율 >= 75% + 해소율 >= 80% + critical 잔존 없음

#### ideate 모드 — 판정 없음

- 수렴/합의 판정을 하지 않는다
- 대신 각 라운드에서 새로 등장한 고유 아이디어 수를 추적
- 라운드 요약에 "아이디어 풀 누적 현황"을 포함
- MAX_ROUNDS 소진 시 전체 아이디어 풀을 정리해서 보고서 생성

#### solve 모드 — 수렴 판정

**1. 해결책 수렴율 (정량)** — 핵심 접근법 유사도

| 일치 비율 | 판정 |
|---|---|
| >= 75% | 방향 수렴 |
| 50~74% | 부분 수렴 |
| < 50% | 수렴 미달 |

**2. Dealbreaker 잔존 여부 (우선 적용)**
- `dealbreakers`가 하나라도 남으면 수렴 불가 (차단 조건)

**3. Critical 트레이드오프 완화율 (정량)**
```
완화율 = (mitigation 제시된 critical tradeoffs 수) / (전체 critical tradeoffs 수)
```
- >= 0.8 → 충분
- < 0.8 → 다음 라운드 필요

**종료 조건 (AND):** dealbreaker 없음 + 수렴율 >= 75% + 완화율 >= 80%

---

### STEP 3 — 라운드 요약 (오케스트레이터)

종료 조건 미충족 시, 다음 라운드를 위해 오케스트레이터가 요약을 생성한다.

#### review 모드 요약
```
PREVIOUS_ROUND_SUMMARY:
  합의된 항목: [모든 에이전트가 agree한 항목]
  잔류 쟁점: [disagree/neutral 있는 항목 + 새 critical/major issues]
  에이전트별 핵심 주장:
    [AGENT]: recommendation + rationale 요약 1-2문장
```

#### ideate 모드 요약
```
PREVIOUS_ROUND_SUMMARY:
  아이디어 풀 (누적): [전체 고유 아이디어 목록 — 제목 + 제안자]
  이번 라운드 신규: [이번 라운드에서 처음 등장한 아이디어]
  결합 아이디어: [combinations에서 생성된 아이디어]
  미탐색 방향: [unexplored_angles 취합]
```

#### solve 모드 요약
```
PREVIOUS_ROUND_SUMMARY:
  해결책 풀:
    [Agent A]: [solution.title] — [description 요약 1문장]
    [Agent B]: ...
  합의 요소: [모든 에이전트의 willing_to_adopt 교집합]
  미결 트레이드오프: [critical + mitigation null인 항목]
  Dealbreakers: [잔존 dealbreaker 목록]
```

---

### STEP 4 — 최종 보고서

종료 조건 달성 또는 MAX_ROUNDS 소진 시 최종 보고서를 생성한다.

#### review 모드 보고서

```markdown
# Debate 최종 보고서 — Review

**분석 대상:** [대상 요약]
**참여 에이전트:** [목록]
**진행 라운드:** [N] / [MAX_ROUNDS]
**합의 상태:** 합의 달성 | 부분 합의 | 합의 미달

---

## 1. 최종 권고
**합의된 권고:** [approve | approve_with_conditions | reject | 불일치]
[핵심 근거 요약 2-4문장]

## 2. 합의된 항목
- [항목 1]
- [항목 2]

## 3. 잔류 이견
| 쟁점 | 심각도 | 에이전트별 입장 |
|---|---|---|
| [내용] | critical/major/minor | [A: agree, B: disagree, ...] |

## 4. 에이전트별 최종 입장
### [Agent A]
- **권고:** approve/reject/...
- **핵심 발견:** ...
- **주요 우려:** ...

## 5. 조건부 승인 시 권고 조치 (해당 시)
1. [필수 조치]

---
*생성일시: [timestamp] | 스킬: debate (review)*
```

#### ideate 모드 보고서

```markdown
# Debate 최종 보고서 — Ideate

**탐색 주제:** [대상 요약]
**참여 에이전트:** [목록]
**진행 라운드:** [N] / [MAX_ROUNDS]
**생성된 고유 아이디어 수:** [N]개

---

## 1. 아이디어 맵

### 카테고리별 정리
#### [카테고리/태그 A]
- **[아이디어 제목]** (by [Agent]) — [설명 1문장] | novelty: high/medium/low
- ...

#### [카테고리/태그 B]
- ...

## 2. 주목할 결합 아이디어
| 원본 아이디어들 | 결합 결과 | 시너지 |
|---|---|---|
| [A의 X + B의 Y] | [결합 아이디어] | [시너지 설명] |

## 3. 미탐색 방향
> 에이전트들이 제안했으나 아직 깊이 다루지 못한 각도
- [방향 1]
- [방향 2]

## 4. 에이전트별 관점 요약
### [Agent A]
- **고유 관점:** ...
- **제안 아이디어 수:** N개
- **가장 높은 novelty 아이디어:** ...

---
*생성일시: [timestamp] | 스킬: debate (ideate)*
```

#### solve 모드 보고서

```markdown
# Debate 최종 보고서 — Solve

**문제:** [문제 요약]
**참여 에이전트:** [목록]
**진행 라운드:** [N] / [MAX_ROUNDS]
**수렴 상태:** 수렴 달성 | 부분 수렴 | 수렴 미달

---

## 1. 합의된 해결책
**제목:** [통합 해결책 제목]
[핵심 접근법 3-5문장 서술]

### 실행 단계
1. [단계 1]
2. [단계 2]

### 실현 가능성
**종합 평가:** high | medium | low
[근거 2-3문장]

## 2. 합의된 핵심 요소
- [요소 1]
- [요소 2]

## 3. 트레이드오프 및 완화 방안
| 트레이드오프 | 심각도 | 완화 방안 |
|---|---|---|
| [내용] | critical/major/minor | [방안 또는 "미결"] |

## 4. 미해소 분기점 (해당 시)
| 쟁점 | 에이전트별 입장 | 오케스트레이터 판단 |
|---|---|---|
| [분기점] | [A: 안X, B: 안Y] | [판단] |

## 5. 에이전트별 최종 제안
### [Agent A]
- **제안:** [title]
- **핵심 접근:** ...
- **실현 가능성:** high/medium/low

## 6. 다음 단계 권고
1. [즉각 조치 1]
2. [즉각 조치 2]

---
*생성일시: [timestamp] | 스킬: debate (solve)*
```

---

## 오케스트레이터 체크리스트

### 매 라운드 시작 전
- [ ] 분석 대상이 subagent 프롬프트에 완전히 포함됐는가
- [ ] 이전 라운드 요약이 정확하게 구성됐는가
- [ ] 각 subagent가 **독립적으로** spawn됐는가 (동일 컨텍스트 공유 금지)
- [ ] Prism instrument 지시문이 해당 subagent에 올바르게 삽입됐는가

### 매 라운드 종료 후
- [ ] JSON 파싱 실패한 응답은 해당 에이전트 결과 제외 후 사용자에게 고지
- [ ] 모드별 판정 로직 올바르게 적용 (ideate는 판정 없음)
- [ ] review/solve: 차단 조건(critical 쟁점 / dealbreaker) 먼저 확인
- [ ] review/solve: 정량 지표 계산 후 종료 조건 판단

### 최종 보고서 작성 시
- [ ] 모드에 맞는 보고서 포맷 사용
- [ ] MAX_ROUNDS 소진으로 종료된 경우 미해소 항목 명시
- [ ] 모드 전환이 자연스러운 경우 사용자에게 다음 단계 제안 (강제 아님)

---

## NOT this skill

- **카탈로그 탐색** — `/prism search`로 라우팅
- **instrument 생성** — `/prism`으로 라우팅
- **instrument 로드** — `/prism fetch`로 라우팅 (debate 내에서 자동 호출 가능)
- **1명의 분석가가 여러 도구를 순차 적용하는 분석** — Prism의 기본 7단계 워크플로우를 직접 사용. debate는 **다수 관점의 병렬 분석 + 합의/발산**이 필요할 때만 사용

---

## 운영 규칙

- subagent는 반드시 Agent tool로 **병렬 spawn** — 순차 실행 금지
- 각 subagent는 독립 컨텍스트 — 다른 subagent의 현재 라운드 응답을 실시간 공유하지 않음
- 라운드 간 정보 전달은 오케스트레이터가 생성한 요약문으로만 수행
- MAX_ROUNDS 기본값 4 — 대부분의 토론은 2-3라운드에 수렴. 4라운드는 안전 상한
- JSON 파싱 실패 시 해당 에이전트는 해당 라운드에서 제외하고, 사용자에게 한 줄 고지
