# Forge Description 재설계 Research Report

> Created: 2026-03-08
> Issue: description 키워드 무한 증가 문제 해결 — 의미 기반 트리거로 전환 + name v5 제거
> Scale: small
> Type: code

## 1. Current State Analysis

현재 forge SKILL.md의 frontmatter description은 **1,294자 / 149단어 / 15줄**로 구성되어 있다.

내용 구성:
- 목적 설명 (30%): 언제 사용하는지, 무엇을 하는지
- 구현 상세 (50%): --direct, --from, 키워드 리스트 등 런타임 옵션
- 절대 규칙 (20%): 컴팩트 후 보존용 규칙 5개

문제의 근본 원인: 키워드가 부족해서 트리거가 안 된다고 판단 → 키워드 추가 → description 무한 증가 패턴.

## 2. Findings (by severity)

### HIGH

#### H1. 스킬 트리거링은 키워드 매칭이 아닌 의미(Semantic) 기반이다
- **Location:** Claude Code 스킬 시스템 아키텍처
- **Description:** skill-creator 공식 문서에 명시: "Skills appear in Claude's available_skills list with their name + description, and Claude decides whether to consult a skill based on that description." Claude는 사용자 요청과 description 간의 **의미적 관련성**을 평가하여 트리거 여부를 결정한다. 키워드 매칭 알고리즘이 아님.
- **Impact:** 키워드 나열 방식은 오히려 역효과 — Claude가 의미적 판단 대신 키워드 존재 여부에 의존하게 유도할 수 있다. 키워드에 없는 표현("건차를 다시 살수있지만...")은 매칭되지 않는다.
- **Evidence:** Eval 0("React와 Express로 블로그 앱 만들어줘")는 키워드 리스트에 없는 표현이지만 의미적으로 "복잡한 멀티파일 개발"에 해당. 키워드 기반으로는 놓칠 수 있지만 의미 기반으로는 잡힌다.

#### H2. Description에 런타임 규칙이 포함되어 트리거 신호를 희석시킨다
- **Location:** SKILL.md frontmatter 라인 15-18 (절대 규칙 섹션)
- **Description:** 절대 규칙 5개가 description에 포함되어 있지만, 이들은 **트리거 결정과 무관한 런타임 제약**이다. 동일한 규칙이 SKILL.md 본문(라인 28-42)에 이미 상세히 존재한다.
- **Impact:** description의 30%가 트리거와 무관한 내용으로 점유 → Claude가 의미 매칭에 활용할 신호가 희석. description이 길수록 핵심 트리거 정보의 비중이 줄어든다.
- **Evidence:** 다른 잘 설계된 스킬(pdf, xlsx, docx)은 description에 런타임 규칙을 넣지 않는다. 오직 "언제 사용하는지"와 "무엇을 하는지"만 포함.

#### H3. "푸시(Pushy)" 패턴이 부족하다
- **Location:** 현재 description 전체
- **Description:** skill-creator 문서에 따르면, Claude는 스킬을 **과소 트리거(undertrigger)**하는 경향이 있다. 이를 방지하려면 description을 적극적으로 작성해야 한다: "Make sure to use this skill whenever..." 패턴. 현재 forge description은 카테고리를 나열할 뿐, 적극적으로 사용을 유도하지 않는다.
- **Impact:** 복잡한 개발 요청인데도 forge가 트리거되지 않는 경우 발생 (건차 케이스)
- **Evidence:** skill-creator 문서: "Currently Claude has a tendency to 'undertrigger' skills -- to not use them when they'd be useful. To combat this, please make the skill descriptions a little bit 'pushy'."

### MEDIUM

#### M1. 키워드 리스트가 실제 사용 패턴과 불일치한다
- **Location:** SKILL.md 라인 10 (자동 감지 키워드)
- **Description:** "멀티파일, 멀티스텝, 기능추가, 전면 수정, refactor, build, implement, 처리, 추가, 개발, 변경, 적용, 작업, ~하게 해줘, ~되도록" — 72+ 문자의 키워드 리스트. 실제 사용자는 "블로그 앱 만들어줘", "JWT 인증 포함된 REST API 만들어줘" 같은 자연어를 사용한다.
- **Impact:** 키워드에 없는 표현은 놓치고, 키워드에 있지만 단순한 요청("처리해줘")은 잘못 트리거할 수 있다.

#### M2. Description 옵션 상세가 body와 중복된다
- **Location:** SKILL.md 라인 7-9 vs 본문 라인 88-100
- **Description:** --direct, --from 옵션 설명이 description과 body에 모두 존재. 이는 구현 상세지 트리거 신호가 아니다.
- **Impact:** 불필요한 description 비대화

### LOW

#### L1. Name에 "v5" 버전 번호가 포함되어 있다
- **Location:** SKILL.md 라인 2 (`name: Forge v5`)
- **Description:** 스킬 이름에 버전 번호가 있으면 사용자 혼란 유발. "Forge"로 통일해야 한다.
- **Impact:** 사소하지만 일관성 저하

## 3. Related Code Structure

```
SKILL.md frontmatter (description)
  → Claude 스킬 목록에 항상 노출 (컴팩트 후에도)
  → 트리거 결정의 유일한 입력

SKILL.md body
  → 트리거 후 로드됨
  → 절대 규칙, 실행 플로우, 상세 옵션 포함
  → 컴팩트 시 손실 가능 → 자가 복구 메커니즘 존재
```

## 4. Recommended Approach & Patterns

### 접근법: 의미 기반 트리거 description 재설계

**원칙:**
1. **트리거 전용**: description은 "이 스킬을 사용할지 말지"를 결정하는 신호만 포함
2. **푸시 패턴**: "반드시 이 스킬을 사용하세요" 형태의 적극적 유도
3. **의미 범주**: 키워드 나열 대신 의미적 범주로 트리거 (예: "여러 파일에 걸친 개발 작업")
4. **런타임 분리**: 절대 규칙은 body에만 유지 (자가 복구 메커니즘이 이미 존재)

**참고 패턴 (skill-creator 권장):**
```
[무엇을 하는지 1문장] + [언제 사용하는지 "Use when..." 패턴] + [적극적 트리거 유도]
```

**목표 크기:** 300-500자 (현재 1,294자 → 60-75% 감축)

**절대 규칙 보존 전략:**
- description에서 제거하되, body의 자가 복구 섹션이 이미 "Step 진입 시 SKILL.md를 다시 읽어라"고 지시함
- 컴팩트 후에도 description은 항상 컨텍스트에 남으므로, description에 "컨텍스트 손실 시 SKILL.md 재로드" 한 줄만 남기면 충분
