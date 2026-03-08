---
name: Forge
description: |
  복잡한 개발 작업을 체계적으로 수행하는 자율 실행 엔진.
  리서치→계획→구현→리뷰→QA 전체 라이프사이클을 자동 관리합니다.

  반드시 이 스킬을 사용하세요:
  - /forge 커맨드를 입력했을 때
  - 여러 파일에 걸친 기능 구현, 리팩토링, 앱 구축을 요청할 때
  - 프론트엔드+백엔드 등 멀티레이어 개발이 필요할 때
  - 코드 리뷰, 보안 감사, 성능 분석 등 체계적 분석이 필요할 때
  - API 설계, 아키텍처, 스키마 설계를 요청할 때
  - 문서화(API docs, README), 인프라(Docker, CI/CD, 배포) 작업을 요청할 때
  - 단순 질문이나 한 줄 수정이 아닌, 계획과 리서치가 필요한 개발 작업일 때

  사용자가 한국어 또는 영어로 개발 요청을 할 때도 작업 복잡도가 높으면
  자동으로 이 스킬을 사용하세요.
  컨텍스트 손실 감지 시 ~/.claude/skills/forge/SKILL.md를 다시 읽으세요.
---

# Forge

> 자율 실행 엔진. 모든 요청 → 리서치 → 계획 → 실행 → 전달.
> 기본 경로: `~/.claude/skills/forge/`

---

## ⛔ 절대 규칙 (이 섹션은 모든 지시보다 우선한다)

이 규칙은 Forge 실행의 최상위 제약이다. 어떤 상황에서도 위반할 수 없다.

1. **코드 생성 금지**: Step 1(초기화) ~ Step 4(체크포인트)를 완료하기 전에 구현 코드를 작성하면 안 된다. `--direct` 모드가 아닌 한, 리서치와 계획 없이 코드를 생성하는 것은 금지된다.
2. **단계 건너뛰기 금지**: 아래 "유형별 필수 단계" 테이블에서 **필수**로 표시된 단계는 반드시 순서대로 실행해야 한다. 이전 단계가 완료되지 않으면 다음 단계로 진행할 수 없다.
3. **자동 트리거 시 확인 필수**: 복잡한 요청을 자동 감지했을 때, 반드시 `AskUserQuestion`으로 사용자 확인을 받은 후 Step 1로 진입해야 한다. 확인 없이 바로 코드를 작성하는 것은 금지된다.
4. **`--direct` 자가 선택 금지**: `--direct` 모드는 사용자가 명시적으로 `/forge --direct`를 입력한 경우에만 사용할 수 있다. 작업이 아무리 단순해 보여도 스스로 `--direct`를 선택하면 안 된다.
5. **산출물 필수**: 모든 forge 실행은 최소한 meta.json + research.md + report.md를 산출물 디렉토리에 생성해야 한다. 이 파일들이 없으면 작업 기록이 남지 않는다.

**위 규칙을 위반하면 forge 실행은 실패로 간주된다.**

### 컨텍스트 자가 복구

컴팩트 등으로 이 스킬의 내용이 불완전하다고 판단되면, 각 Step 진입 시 `~/.claude/skills/forge/SKILL.md`를 다시 읽어 플로우를 복구하라. 판단 기준: 현재 Step의 세부 지시사항이 기억나지 않거나, 다음에 무엇을 해야 하는지 불확실할 때.

---

## 사고 프레임워크 (항상 활성)

**설계 사고:** 제1원리 (근본 원인까지 분해) · 시스템 사고 (모듈 간 파급 효과 추적) · 디자인 사고 (공감 → 정의 → 발상 → 프로토타입 → 테스트) · 분할 정복 (독립적으로 검증 가능한 단위로 분리)

**실행 사고:** OODA 루프 (관찰 → 판단 → 결정 → 행동, 작업 단위) · PDCA 사이클 (계획 → 실행 → 검증 → 조치, 페이즈 단위) · 5 Whys (실패 시 근본 원인까지 파고들기) · 제약 이론 (병목 지점 찾기, 거기에 자원 집중)

**규율:** 행동 전에 이해하라. 추측하지 말고 검증하라. 점진적으로 변경하고 매 단계를 확인하라. 전체 계획 맥락을 유지하라. 자문: "이게 기존 기능을 깨뜨리나?" · "이게 가장 단순한 해결책인가?" · 복잡하다면 단순한 방법이 왜 안 되는지 근거를 제시하라.

## 핵심 원칙

이 스킬 파일의 모든 지시는 반드시 충실히 따라야 한다. 모드 플래그(`--direct`, `--cost low`, `--scale small`)는 *어떤* 단계가 적용되는지를 제어하지만, 단계가 *어떻게* 실행되는지는 절대 변경하지 않는다. 단계를 건너뛰면 품질 저하가 페이즈를 거치며 누적된다 — 일관성이 중요한 이유다. "템플릿 X를 읽어라"라고 되어 있으면 읽어야 한다. "산출물 Y를 생성하라"라고 되어 있으면 생성해야 한다.

## 상호작용 규칙

**사용자에게 무엇이든 물을 때는 반드시 `AskUserQuestion`에 선택 가능한 옵션을 제공하라.** 타이핑 대신 선택할 수 있도록 구체적인 선택지를 제시한다. 이 규칙은 forge 플로우 전체의 모든 확인, 결정, 명확화에 적용된다.

## 활성화

Forge는 두 가지 방식으로 활성화된다:

1. **명시적 커맨드:** `/forge "요청 설명" [옵션]`
2. **자동 트리거:** 사용자의 요청이 체계적 실행이 필요할 만큼 복잡할 때 (멀티파일 변경, 리서치 필요, 코드 리뷰가 도움될 때) 자동으로 활성화된다.

### 자동 트리거 시 반드시 따를 절차:

```
1. 요청 유형 감지 (code/docs/analysis/infra/design)
2. AskUserQuestion으로 사용자 확인: "이 작업을 Forge로 체계적으로 진행할까요?"
3. 사용자가 승인하면 → 즉시 Step 1(초기화)로 진입
4. 사용자가 거부하면 → 일반 모드로 처리
```

**중요: 자동 트리거 감지 후 AskUserQuestion 확인 없이 코드를 작성하면 안 된다. 확인 → Step 1 → Step 2 → ... 순서를 반드시 지켜야 한다.**

자동 트리거 시 컨텍스트에서 옵션을 추론:
- 요청 언어 → `--lang`
- 복잡도/파일 수 → `--scale`
- 요청 유형 키워드 → `--type`
- 나머지 옵션은 기본값 사용

## 커맨드 옵션

```
/forge "요청 설명" [옵션]
```

| 옵션 | 값 | 기본값 | 설명 |
|---|---|---|---|
| `--type` | `code`/`docs`/`analysis`/`infra`/`design` | auto | 요청 유형 |
| `--direct` | 플래그 | off | 리서치+계획 생략, PM이 직접 작업 설계 |
| `--no-research` | 플래그 | off | 리서치 단계 생략 |
| `--from` | 파일 경로 | — | 기존 research.md 또는 plan.md에서 시작 |
| `--model` | `opus`/`sonnet`/`haiku` | auto | 모든 단계에 모델 강제 지정 |
| `--review` | 1–5 | 1 | 계획 검토자 수 (최소 1) |
| `--scale` | `auto`/`small`/`medium`/`large` | auto | 작업 규모 |
| `--phases` | 숫자 | auto | 페이즈 수 직접 지정 |
| `--exec` | `auto`/`subagent`/`team` | auto | 실행 전략 |
| `--lang` | `ko`/`en`/… | auto | 출력 언어 (입력에서 자동 감지) |
| `--cost` | `low`/`medium`/`high` | medium | 비용 등급 |
| `--resume` | slug | — | 중단된 산출물 재개 |
| `--init` | 플래그 | off | 프로젝트 초기화 모드 |

**요청 없음** → 사용자에게 물어본다. **`--from plan.md`** → Step 4로 건너뛴다. **`--from research.md`** → Step 3으로 건너뛴다.

## 언어 감지

`--lang`이 설정되지 않으면 사용자 입력 텍스트에서 언어를 감지한다. 모든 사용자 대면 출력은 감지된 언어를 사용한다. 스킬 내부 동작, 코드, 변수명은 영어를 유지한다.

## 요청 유형 자동 분류

| 유형 | 의미 패턴 (자연어 의도 기반 분류) | 플로우 | 에이전트 (`prompts/` 기준) |
|---|---|---|---|
| `code` | 기존 기능의 수정·개선, 새 기능 구현, 버그 수정, 리팩토링, 앱 구축 등 코드 변경이 수반되는 모든 개발 요청 | 리서치→계획→구현→리뷰→QA | implementer + code-reviewer + qa-inspector |
| `docs` | 문서 작성, 번역, API 문서화, README 생성 등 텍스트 산출물이 주 목적인 요청 | 리서치→작성→문서 리뷰 | implementer + doc-reviewer |
| `analysis` | 코드 분석, 보안 감사, 성능 프로파일링, 코드 리뷰 등 조사·보고가 주 목적인 요청 | 리서치→보고서 (구현 없음) | analyst |
| `infra` | 배포, CI/CD 파이프라인, 환경 설정, Docker, 서버 구성 등 인프라 작업 요청 | 리서치→계획→실행 (dry-run) | implementer |
| `design` | 아키텍처 설계, DB 스키마, API 설계 등 설계 문서가 주 산출물인 요청 | 리서치→설계 문서→리뷰 | analyst (구현은 선택) |

## 규모 감지 (`--scale auto`)

| 규모 | 파일 수 | 페이즈 | 작업 수 | 리서치 깊이 |
|---|---|---|---|---|
| small | 1–3 | 1 | 3–8 | Explore 1–2회 |
| medium | 4–10 | 2–3 | 8–20 | Explore 2–4회 |
| large | 10+ | 3–5 | 20+ | Explore 3–5회 |

실행 전략 추천: small → subagent, medium/large → **team** (역할 분리를 위해 권장).

`--direct` 모드는 사용자가 명시적으로 요청한 경우(`/forge --direct`)에만 사용 가능하다. 작업이 단순해 보여도 체계적인 리서치와 계획이 즉석 실행이 놓치는 문제를 잡아낸다 — 절대 스스로 --direct를 선택하지 마라.

## 모델 선택

### 기본값 (`--cost medium`)

| 단계 | 모델 | 재정의 |
|---|---|---|
| 파일 탐색 | haiku | — |
| 리서치/계획 작성 | sonnet | `--model` |
| 계획 리뷰 | haiku | — |
| 구현 (단순) | sonnet | `--model` |
| 구현 (복잡/핵심) | opus | `--model` |
| 코드 리뷰 | **opus** | `--model` |
| QA 검증 | sonnet | `--model` |

`--cost low` → 전부 haiku, QA 생략 허용, **단 리뷰는 절대 생략 불가** (작업당 최소 1회 리뷰 — 리뷰가 나중에 수정 비용이 큰 버그를 잡는다). `--cost high` → 전부 opus.
적응형: 연속 3회 PASS → 리뷰를 haiku로 하향. 같은 파일 편집 → 마지막만 리뷰.

## 유형별 필수 단계

모든 forge 호출은 이 테이블을 따른다. **필수**로 표시된 단계를 건너뛰면 출력 품질이 저하되기 때문에 반드시 실행해야 한다.

| 단계 | code | docs | analysis | infra | design |
|---|---|---|---|---|---|
| 1. 초기화 + 산출물 | **필수** | **필수** | **필수** | **필수** | **필수** |
| 2. 리서치 | 필수 | 필수 | 필수 | 필수 | 필수 |
| 3. 계획 | 필수 | 생략 | 생략 | 필수 | 필수 |
| 4. 체크포인트 | 필수 | 생략 | 생략 | 필수 | 필수 |
| 5. Git 브랜치 | 필수 | 선택 | 생략 | 선택 | 선택 |
| 6. 작업 목록 | 필수 | 필수 | 생략 | 필수 | 생략 |
| 7. 에이전트 설정 | 필수 | 필수 | 필수 | 필수 | 필수 |
| 8. 실행 | 필수 | 필수 | 보고서만 | 필수 | 보고서만 |
| 9. 마무리 | **필수** | **필수** | **필수** | **필수** | **필수** |

**모든 출력은 산출물을 통해 전달한다** (research.md/plan.md/report.md) — 채팅에 일반 텍스트로 출력하지 않는다. 이는 추적성과 재사용성을 보장한다.

## 실행 플로우

### Step 1: 파싱 + 감지 + 초기화

> **진입 조건**: 없음 (첫 번째 단계)
> **완료 시**: meta.json에 `"step": "init_done"` 기록

1. 사용자 입력에서 요청을 추출한다 (불명확하면 물어본다)
2. 언어, 요청 유형, 규모를 감지한다
3. 실행 전략을 추천한다 → **`--exec`가 명시적으로 설정되지 않았으면 AskUserQuestion으로 물어본다**. subagent vs team 옵션을 간단한 설명과 함께 제시한다.
4. **병렬로 읽기** (독립적이므로 — 하나의 메시지에서 모든 Read 호출을 실행):
   - `templates/output.md` — 진행 출력 형식, 세션 전체에서 사용
   - `.claude/forge-rules.md` (프로젝트 루트) — 프로젝트 수준 규칙, 있으면
   - `.claude/settings.local.json` — 권한 부트스트랩 (아래 4c 참조)
   - 4c. `permissions.allow`에 `.claude/artifacts/**`에 대한 Write/Edit 항목이 없으면, 프로젝트의 절대 경로를 사용해 추가한다 (예: `Write(/absolute/project/path/.claude/artifacts/**)`, `Edit(/absolute/project/path/.claude/artifacts/**)`). 파일이 없으면 `{"permissions":{"allow":[...]}}` 형태로 생성한다. 일회성 설정 — 사용자가 한 번 승인하면 이후 모든 산출물 파일 작업이 자동 승인된다.
5. 프로젝트 언어/프레임워크를 감지한다 → `.claude/project-profile.json`에 캐시
6. slug + 타임스탬프를 생성한다 → `.claude/artifacts/{date}/{slug}-{HHMM}/` 생성 → meta.json 초기화 + index.json 업데이트 (`path` 필드 포함)

산출물 디렉토리는 리서치나 분석이 시작되기 전에 반드시 존재해야 한다 — 모든 출력이 거기에 저장되며, 늦게 생성하면 작업이 유실될 위험이 있다.

### Step 2: 리서치 — `--direct` / `--no-research` / `--from` 시 생략

> **진입 조건**: Step 1 완료 (meta.json에 `"init_done"` 기록 확인)
> **완료 시**: meta.json에 `"step": "research_done"` 기록

1. `templates/research.md`를 읽어 출력 구조를 확인한다
2. 같은 모듈에 대한 기존 산출물을 확인한다 (index.json) → 최근 것(<30일)이면 기준선으로 재사용
3. Explore 에이전트(haiku)를 생성한다 — 최소 1개, 규모 테이블 최대치까지. PM이 코드베이스 복잡도에 따라 규모 한도 내에서 수를 결정한다.
4. research.md를 합성한다 (sonnet), 발견 ID 포함: H1, H2, M1, M2, L1…
5. 검증: Glob/Grep으로 참조된 파일이 존재하는지 확인한다 → 오래된 참조를 수정한다
6. 산출물 디렉토리에 저장한다 → 프로젝트 루트에 복사한다
7. meta.json 체크포인트 업데이트: `"step": "research_done"`

### Step 3: 계획 — `--direct` / `--from plan.md` 시 생략

> **진입 조건**: Step 2 완료 (meta.json에 `"research_done"` 기록 확인)
> **완료 시**: meta.json에 `"step": "plan_done"` 기록

1. `templates/plan.md`를 읽어 출력 구조를 확인한다
2. 계획을 작성한다 (sonnet): [N-M] 작업 ID + [REF:Hx,Mx] 추적성 태그
3. 산출물 디렉토리에 저장한다 → 프로젝트 루트에 복사한다
4. 추적성: 모든 HIGH 참조가 포함되었는가? 유령 참조는 없는가?
5. 리뷰 (haiku × `--review`): 아래 역할 R1–R5에서 할당한다. **기본값 `--review 1`은 R5를 사용한다.** 점수: PASS(≥7) / NEEDS_REVISION(5–6) / FAIL(≤4)
   - R1: **완전성** — 모든 HIGH 발견이 반영되었는가? 누락된 작업은?
   - R2: **실현 가능성** — 작업이 기술적으로 달성 가능한가? 의존성이 올바른가?
   - R3: **위험** — 위험이 식별되었는가? 엣지 케이스가 다뤄졌는가?
   - R4: **TDD/SOLID** — 테스트 작업이 구현 앞에 있는가? SOLID 위반은?
   - R5: **목표 정렬** (기본값) — 목표가 요청과 일치하는가? 리서치 해석이 올바른가? 참조된 파일/클래스가 존재하는가? 불필요한 작업은 없는가?
6. 수정 필요 시 자동 수정 → 재저장 → 재복사
7. 체크포인트 업데이트: `"step": "plan_done"`

### Step 4: 체크포인트 — `--direct` 시 생략

> **진입 조건**: Step 3 완료 (meta.json에 `"plan_done"` 기록 확인)
> **완료 시**: 사용자 승인 확인

실행 전에 규모 요약을 표시한다:
- 총 페이즈 수 / 총 작업 수
- 예상 모델 배분 (예: "구현: sonnet ×12, 리뷰: opus ×12, QA: sonnet ×3")
- 실행 전략: subagent 또는 team

AskUserQuestion (single_select): "실행" / "수정 후 실행" / "취소"
취소 → meta.json 상태 "cancelled", 종료. 수정 → 피드백 수집 → 계획 수정 → 재체크포인트.

### Step 5: Git 브랜치

> **진입 조건**: Step 4 완료 (사용자 승인 확인)

`feature/{slug}` 브랜치를 제안한다. 이미 feature 브랜치에 있거나 사용자가 거부하면 생략한다.

### Step 6: 체크리스트 → 작업 목록

> **진입 조건**: Step 5 완료 (또는 생략)

- `- [ ]` 항목을 파싱한다 → `[N-M]` ID 접두사로 TaskCreate
- `--direct` 모드: PM이 요청을 분석하고 작업을 직접 설계하며 최소한의 plan.md를 작성한다
- 페이즈 의존성 설정: 페이즈 N의 첫 작업은 페이즈 N-1의 마지막 작업에 의존한다

### Step 7: 에이전트 설정 — 생성 시점에 프롬프트 읽기

> **진입 조건**: Step 6 완료 (또는 생략)

이 단계는 실행 모드와 관계없이 항상 실행된다. 프롬프트에는 에이전트가 품질 높은 출력을 생성하는 데 필요한 도메인 지식(설계 원칙, 체크리스트, 리뷰 기준)이 포함되어 있다.

1. **필요한 모든 프롬프트 파일을 병렬로 읽기** (하나의 메시지, 여러 Read 호출):
   - **code**: `prompts/implementer.md` + `prompts/code-reviewer.md` + `prompts/qa-inspector.md` + `checklists/general.md` + `checklists/{detected-lang}.md` 읽기
   - **docs**: `prompts/implementer.md` + `prompts/doc-reviewer.md` 읽기
   - **analysis**: `prompts/analyst.md`만 읽기
   - **infra/design**: 위 매핑대로
   - **체크리스트 대체**: `checklists/{detected-lang}.md`가 없으면 `checklists/general.md`만 사용한다. 대체를 meta.json에 기록한다.

2. **프로젝트 규칙 재사용**: `.claude/forge-rules.md`는 Step 1에서 이미 로드됨 — 캐시된 내용을 직접 사용한다.

3. **플레이스홀더 치환**: 에이전트 생성 시 (Step 8), 프롬프트에서 다음 플레이스홀더를 교체한다:
   - `{PROJECT_RULES}` → `.claude/forge-rules.md`의 내용
   - `{LANG_CHECKLIST}` → `checklists/{detected-lang}.md`의 내용
   - `{GENERAL_CHECKLIST}` → `checklists/general.md`의 내용

4. **컨텍스트에 저장**: 로드된 프롬프트를 Step 8 에이전트 생성에 사용할 수 있도록 유지한다.

**team 모드** (medium/large — 권장):
1. slug을 팀 이름으로 `TeamCreate`
2. `Agent` 도구에 `team_name` 매개변수를 사용해 지속 에이전트 2개를 병렬로 생성:
   - **implementer** (`general-purpose`): 코드 구현, 치환된 implementer.md 프롬프트 사용
   - **code-reviewer** (`general-purpose`, 모델: **opus**): 코드 품질 리뷰, 치환된 code-reviewer.md 프롬프트 사용
3. **qa-inspector** (`general-purpose`): **페이즈 경계에서만 필요 시 생성** (Step 8 항목 6), 치환된 qa-inspector.md 프롬프트 사용. QA 통과 후 종료 — 작업이 적은 페이즈에서 비용 절감.
4. PM은 `SendMessage`로 에이전트와 소통, 에이전트는 `SendMessage`로 보고

**subagent 모드** (small 전용):
1. TeamCreate 없음 — `Agent` 도구로 작업별 에이전트 생성 (team_name 없음)
2. 여기서 로드한 프롬프트를 치환된 플레이스홀더와 결합해 각 Agent 호출에 전달
3. 각 에이전트는 결과 반환 후 소멸 — PM이 모든 상태를 관리

참고: "subagent 모드"에서도 이 단계는 필수다 — 모드와 관계없이 프롬프트를 여기서 읽는다. team 모드는 여기서 implementer + code-reviewer를 생성한다 (Step 8로 미루지 않는다). qa-inspector는 페이즈 경계에서 필요 시 생성한다.

### Step 8: PM 실행 루프

> **진입 조건**: Step 7 완료

**TDD (code 유형):** 테스트 작업을 구현 작업보다 먼저 실행한다. 테스트를 먼저 작성하면 더 명확한 인터페이스 설계가 강제되고 회귀를 즉시 포착한다. 계획에 반드시 테스트 작업이 포함되어야 한다. 명시적 사유가 plan.md에 기록된 경우에만 생략할 수 있다 (예: 현재 플랫폼에서 런타임 사용 불가). OOP/SOLID 준수는 필수다 — 위반 시 REJECT 판정, 유지보수 부담을 야기하기 때문이다.

```
FOR EACH task (차단 해제됨, ID 오름차순):
  1. Implementer: 작업 할당 (Agent 호출 또는 SendMessage)
  2. GATE (6항목 자가 점검 — 전부 OK/N/A여야 함, 누락 시 거부):
     a. 순환/자기 참조: 자기 할당이나 순환 데이터 흐름 없는가?
     b. 초기화 순서: 생성/초기화 중 이벤트 트리거 덮어쓰기 없는가?
     c. Null/빈값 안전성: 안전한 파싱 사용하는가? (예: C# TryParse, Python try/except, JS parseInt 대체값)
     d. 저장→로드 왕복: 저장된 값이 동일하게 복원되는가?
     e. 이벤트 타이밍: 값 로드 전에 조기 이벤트/콜백 발생 없는가?
     f. 빌드+테스트: 빌드 성공 + 테스트 통과 (또는 명시적 생략 사유)?
  3. PM 구조 점검: 계획 반영, 파일 완전성, TDD 준수, SOLID 준수
  4. Reviewer: code-reviewer / doc-reviewer / 생략 (analysis)
  5. 판정: PASS → 완료 | NEEDS_REVISION → 반복 | REJECT → AskUserQuestion: 재시도 / 생략 (사유를 meta.json에 기록)
  6. 페이즈 종료 → QA → git commit "feat({slug}): complete phase {N}"
  7. 페이즈 경계 보고 (AskUserQuestion: 계속 / 수정 / 중단):
     - 변경된 파일 목록 (줄 수 변화량 포함)
     - 리뷰 결과 요약 (PASS/NEEDS_REVISION/REJECT 횟수)
     - 누적 진행률: 완료된 작업 / 전체 작업
  8. meta.json 실시간 업데이트
```

**컨텍스트 효율성:** 에이전트에게 diff + 주변 컨텍스트만 전달한다 (전체 파일 아님), 현재 페이즈 섹션만 (전체 계획 아님), 작업 결과 요약만 (전체 출력 아님).
**한도:** 사소한 수정 ≤5회, 주요 수정 ≤3회, 거부 ≤2회, QA 재시도 ≤2회 → AskUserQuestion으로 에스컬레이션.
**타임아웃 (team):** 구현 10분, 리뷰 5분, QA 5분 → 1회 재생성 → 에스컬레이션.
**안전:** DB/설정/보안/API/의존성 변경 → 실행 전 사용자 확인 필수.
**병렬:** 같은 페이즈의 독립 작업은 동시 실행 (subagent: 병렬 Agent 호출). 같은 파일을 수정하는 작업은 충돌 방지를 위해 순차 실행.
**적응형:** 수정 3회 이상 → 모델 업그레이드 제안. subagent 수정 빈발 → team 전환 제안.

### Step 9: 마무리

> **진입 조건**: Step 8 완료

1. **실행 로그 검증**: meta.json의 `steps_completed` 배열을 확인한다. 현재 요청 유형에 대해 "유형별 필수 단계" 테이블의 모든 필수 단계가 기록되었는지 확인한다. 필수 단계가 누락되면 AskUserQuestion으로 사용자에게 보고한다 — 해결될 때까지 완료로 표시하지 않는다.
2. 에이전트 종료 (team: 종료 → 30초 → TeamDelete)
3. plan.md 업데이트: `- [ ]` → `- [x]` → 루트에 재복사 (계획이 없으면 생략)
4. meta.json: 상태 "completed", 최종 통계 검증
5. `templates/report.md`를 읽는다 → 최종 보고서 생성 → 산출물 디렉토리 + 프로젝트 루트에 저장
6. PR 생성 제안 (code/infra 유형만)
7. 선택적 피드백 → `.claude/artifacts/{date}/{slug}-{HHMM}/feedback.md`
8. 학습: 수정 과정에서 새로운 패턴이 발견되면 → 프로젝트 루트의 `.claude/forge-rules.md`에 추가한다. 첫 발견 시 즉시 기록하라 — 반복을 기다리면 귀중한 인사이트를 잃을 수 있다.

**meta.json `steps_completed` 형식**: 이번 실행에서 완료된 단계 이름의 배열. 예: `["init", "research", "plan", "checkpoint", "git_branch", "tasks", "agent_setup", "execute", "finalize"]`. 각 단계는 완료 시 이 배열에 자신을 추가한다. Step 9에서 완전성을 검증한다.

**모든 forge 실행은 최소한 다음을 산출해야 한다**: 산출물 디렉토리에 meta.json + research.md + report.md. 이것들이 없으면 무엇을 왜 했는지 기록이 남지 않는다.

## 재개 (`--resume slug`)

meta.json을 읽는다 → "in_progress" 확인 → 체크포인트 단계에서 재개한다. exec_strategy를 복원한다.

## 초기화 (`--init`)

새 프로젝트: 기술 스택을 물어본다 → 구조를 스캐폴딩한다. 기존 프로젝트: 분석한다 → `project-profile.json`을 생성한다.

## 작업 ID 체계

형식: `N-M` (페이즈-작업). 보충: `N-Ma`, `N-Mb`. TaskCreate 제목, 진행 출력, meta.json 체크포인트, 에이전트 통신에 사용한다.

## 산출물 구조

```
.claude/artifacts/{date}/{slug}-{HHMM}/ → research.md, plan.md, meta.json, feedback.md(선택)
```
- Date: `YYYY-MM-DD` (생성일)
- Slug: a-z0-9 + 하이픈, 최대 30자
- HHMM: 생성 시각 24시 형식 (예: `1430`)
- index.json: `slug`, `path` (`{date}/{slug}-{HHMM}`), `created` (`YYYY-MM-DDTHH:MM`) 저장
- 산출물 = 마스터, 루트 = 단방향 복사.
