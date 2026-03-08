# Forge Description 재설계 Implementation Plan

> Created: 2026-03-08
> Goal: description 키워드 무한 증가 문제 해결 — 의미 기반 트리거로 전환 + name v5 제거
> Reference: research.md
> Methodology: agile
> Scale: small
> Type: code

## 1. Project Overview

Forge SKILL.md의 frontmatter description을 키워드 나열 방식에서 의미 기반 트리거 방식으로 전환한다.
현재 1,294자/149단어의 description을 300-500자로 감축하되, 트리거 정확도는 유지하거나 향상시킨다.

**범위:**
- SKILL.md frontmatter (name + description) 수정
- SKILL.md body의 "요청 유형 자동 분류" 테이블에서 트리거 키워드 컬럼 재설계
- 본문 제목 "Forge v5" → "Forge" 변경

**제외:**
- SKILL.md body의 실행 플로우 (Step 1-9) 변경 없음
- 절대 규칙 섹션 내용 변경 없음 (body의 것)
- 템플릿/프롬프트/체크리스트 변경 없음

## 2. Technical Stack (with rationale)

- 대상 파일: SKILL.md (YAML frontmatter + Markdown body)
- 테스트: evals.json 기반 수동 검증
- 변경 도구: Edit tool

## 3. Implementation Phases

### Phase 1: Description 재설계 + Name 변경
**Goal:** frontmatter를 의미 기반으로 재설계하고, 본문 일관성 확보
**Completion criteria:**
- description이 300-500자 이내
- 키워드 리스트 제거되고 의미 범주로 대체
- 절대 규칙이 description에서 제거되고 body에만 존재
- name: "Forge" (v5 제거)
- 기존 eval 3개 모두 PASS 가능한 구조

## 4. Task Checklist

### Phase 1: Description 재설계 + Name 변경

- [x] [1-1] name 필드를 "Forge v5" → "Forge"로 변경 [REF:L1]
- [x] [1-2] 본문 제목 "# Forge v5" → "# Forge"로 변경 [REF:L1]
- [x] [1-3] description을 의미 기반 구조로 재작성 [REF:H1,H2,H3]
  - 1문장 목적 설명 + "Use when..." 의미 범주 + 푸시 패턴
  - 키워드 리스트 제거
  - 절대 규칙 제거 (컨텍스트 복구 한 줄만 유지)
  - 결과: ~470자 (목표 300-500자 달성)
- [x] [1-4] 본문 "요청 유형 자동 분류" 테이블의 "트리거 키워드" 컬럼을 "의미 패턴" 컬럼으로 재설계 [REF:M1]
  - 키워드 나열 → 자연어 의도 기반 설명으로 변경
- [x] [1-5] 코드 리뷰: 변경사항 전체 검증 — PASS (8/8 기준 충족) [REF:H1,H2,H3,M1,M2,L1]

## 5. 신규 Description 초안

```yaml
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

  사용자가 한국어 또는 영어로 "~만들어줘", "~구현해줘", "~처리해줘" 같은
  개발 요청을 할 때도 작업 복잡도가 높으면 자동으로 이 스킬을 사용하세요.
  컨텍스트 손실 감지 시 ~/.claude/skills/forge/SKILL.md를 다시 읽으세요.
```

**크기:** ~470자 (현재 1,294자에서 64% 감축)
**구조 변화:**
- 키워드 나열 → 의미 범주 설명
- 런타임 규칙 (절대 규칙 5개) → 제거 (body에만 존재)
- 옵션 상세 (--direct, --from) → 제거 (body에만 존재)
- 푸시 패턴 추가: "반드시 이 스킬을 사용하세요"
- 컨텍스트 복구 한 줄 유지

## 6. Risk & Mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| 의미 범주가 너무 넓어 과잉 트리거 | 단순 질문에도 forge 활성화 | "단순 질문이나 한 줄 수정이 아닌" 제외 조건 명시 |
| 절대 규칙 제거로 컴팩트 후 규칙 유실 | 플로우 무시 | body의 자가 복구 메커니즘이 이미 존재 + description에 복구 지시 1줄 유지 |
| 한국어 의미 범주가 영어 요청 커버 못함 | 영어 사용자 트리거 실패 | "한국어 또는 영어로" 명시 + 영어 키워드 불필요 (의미 매칭이므로) |
