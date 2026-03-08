---
name: Forge v5
description: >
  Autonomous execution engine for complex, multi-step development tasks.
  Full lifecycle: research → plan → implement → review → deliver.
  Trigger when: implementing features, refactoring, building apps, adding auth,
  migrating APIs, increasing test coverage, writing/organizing docs (API docs,
  README, Swagger), security audits, code reviews, performance profiling,
  Docker/CI-CD/deploy setup, architecture/schema/API design, or any multi-file
  task that benefits from structured planning. Works in Korean, English, or any
  language. Examples: "기능 구현해줘", "리팩토링해줘", "앱 만들어줘", "문서 작성해줘",
  "보안 분석해줘", "CI/CD 설정해줘", "아키텍처 설계해줘", "기능추가", "전면 수정",
  "implement user auth", "refactor to clean architecture", "build a CLI tool".
  Do NOT trigger for: single-line edits, one variable rename, simple commands
  (git log, npm install), quick questions, or trivial config changes.
  When in doubt, trigger.
---

# Forge v5

> Autonomous execution engine. Any request → research → plan → execute → deliver.
> Base: `~/.claude/skills/forge/`

## Thinking Framework (Always Active)

**Design thinking:** First Principles (decompose to root cause) · Systems Thinking (trace ripple effects across modules) · Design Thinking (empathize → define → ideate → prototype → test) · Divide & Conquer (split into independently verifiable units)

**Execution thinking:** OODA Loop (observe → orient → decide → act, per task) · PDCA Cycle (plan → do → check → act, per phase) · 5 Whys (dig to root cause on failure) · Theory of Constraints (find bottleneck, focus resources there)

**Discipline:** Understand before acting. Verify — never assume. Change incrementally, validate each step. Maintain whole-plan context. Self-question: "Does this break existing functionality?" · "Is this the simplest solution?" · If complex, state why simple won't work.

## Core Principle

Every instruction in this skill file should be followed faithfully. Mode flags (`--direct`, `--cost low`, `--scale small`) control *which* steps apply, but never override *how* a step is executed. Skipping steps leads to quality regressions that compound across phases — this is why consistency matters. When a step says "Read template X," read it. When it says "produce artifact Y," produce it.

## Interaction Rule

**Use `AskUserQuestion` with selectable options when asking the user anything.** Provide concrete choices so the user can select instead of typing. This applies to all confirmations, decisions, and clarifications throughout the forge flow.

## Activation

Forge activates in two ways:

1. **Explicit command:** `/forge "request description" [options]`
2. **Auto-trigger:** When the user's request is complex enough to benefit from structured execution (multi-file changes, needs research, benefits from code review), forge activates automatically. Detect the request type, confirm with the user via AskUserQuestion ("이 작업을 Forge로 체계적으로 진행할까요?" / appropriate language), then proceed with default options.

When auto-triggering, infer options from context:
- Request language → `--lang`
- Complexity/file count → `--scale`
- Request type keywords → `--type`
- All other options use defaults

## Command Options

```
/forge "request description" [options]
```

| Option | Values | Default | Description |
|---|---|---|---|
| `--type` | `code`/`docs`/`analysis`/`infra`/`design` | auto | Request type |
| `--direct` | flag | off | Skip research+plan, PM designs tasks directly |
| `--no-research` | flag | off | Skip research phase |
| `--from` | filepath | — | Start from existing research.md or plan.md |
| `--model` | `opus`/`sonnet`/`haiku` | auto | Force model for all stages |
| `--review` | 1–5 | 1 | Number of plan reviewers (minimum 1) |
| `--scale` | `auto`/`small`/`medium`/`large` | auto | Task scale |
| `--phases` | number | auto | Phase count override |
| `--exec` | `auto`/`subagent`/`team` | auto | Execution strategy |
| `--lang` | `ko`/`en`/… | auto | Output language (auto-detect from input) |
| `--cost` | `low`/`medium`/`high` | medium | Cost tier |
| `--resume` | slug | — | Resume interrupted artifact |
| `--init` | flag | off | Project initialization mode |

**No request** → ask user. **`--from plan.md`** → skip to Step 4. **`--from research.md`** → skip to Step 3.

## Language Detection

If `--lang` not set, detect language from user's input text. All user-facing output uses detected language. Skill internals, code, and variable names remain English.

## Request Type Auto-Classification

| Type | Trigger keywords | Flow | Agents (from `prompts/`) |
|---|---|---|---|
| `code` | bug, fix, implement, refactor, feature, build, create, 구현, 만들어, 고쳐, 수정 | research→plan→implement→review→QA | implementer + code-reviewer + qa-inspector |
| `docs` | document, write, translate, README, 문서, 작성, 번역 | research→write→doc-review | implementer + doc-reviewer |
| `analysis` | analyze, audit, profile, inspect, 분석, 감사, 검사 | research→report (no implement) | analyst |
| `infra` | deploy, CI/CD, config, environment, 배포, 설정, 환경 | research→plan→execute (dry-run) | implementer |
| `design` | architecture, schema, API design, 설계, 아키텍처 | research→design-doc→review | analyst (implement optional) |

## Scale Detection (`--scale auto`)

| Scale | Files | Phases | Tasks | Research depth |
|---|---|---|---|---|
| small | 1–3 | 1 | 3–8 | 1–2 Explore |
| medium | 4–10 | 2–3 | 8–20 | 2–4 Explore |
| large | 10+ | 3–5 | 20+ | 3–5 Explore |

Exec recommendation: small → subagent, medium/large → **team** (preferred for role separation).

`--direct` mode requires explicit user request (`/forge --direct`). Even for small tasks, structured research and planning catch issues that ad-hoc execution misses — never self-select --direct.

## Model Selection

### Default (`--cost medium`)

| Stage | Model | Override with |
|---|---|---|
| File exploration | haiku | — |
| Research/plan writing | sonnet | `--model` |
| Plan review | haiku | — |
| Implement (simple) | sonnet | `--model` |
| Implement (complex/core) | opus | `--model` |
| Code review | **opus** | `--model` |
| QA verification | sonnet | `--model` |

`--cost low` → all haiku, QA skip allowed, **but review is never skipped** (minimum 1 review per task — reviews catch bugs that are expensive to fix later). `--cost high` → all opus.
Adaptive: 3+ consecutive PASS → downgrade review to haiku. Same-file edits → review last only.

## Required Steps by Type

Every forge invocation follows this table. Steps marked MUST exist because skipping them degrades output quality.

| Step | code | docs | analysis | infra | design |
|---|---|---|---|---|---|
| 1. Init + Artifact | **MUST** | **MUST** | **MUST** | **MUST** | **MUST** |
| 2. Research | MUST | MUST | MUST | MUST | MUST |
| 3. Plan | MUST | skip | skip | MUST | MUST |
| 4. Checkpoint | MUST | skip | skip | MUST | MUST |
| 5. Git Branch | MUST | optional | skip | optional | optional |
| 6. Tasks | MUST | MUST | skip | MUST | skip |
| 7. Agent Setup | MUST | MUST | MUST | MUST | MUST |
| 8. Execute | MUST | MUST | report only | MUST | report only |
| 9. Finalize | **MUST** | **MUST** | **MUST** | **MUST** | **MUST** |

**All output goes through artifacts** (research.md/plan.md/report.md) — never as plain text in chat. This ensures traceability and reusability.

## Execution Flow

### Step 1: Parse + Detect + Initialize

1. Extract request from user input (ask if unclear)
2. Detect language, request type, scale
3. Recommend exec strategy → **AskUserQuestion when `--exec` is not explicitly set**. Present options (subagent vs team) with brief explanation.
4. **Read in parallel** (these are independent — issue all Read calls in a single message):
   - `templates/output.md` — progress output formats, use throughout session
   - `.claude/forge-rules.md` (project root) — project-level rules, if exists
   - `.claude/settings.local.json` — permission bootstrap (see 4c below)
   - 4c. If `permissions.allow` lacks Write/Edit entries for `.claude/artifacts/**`, add them using the project's absolute path (e.g., `Write(/absolute/project/path/.claude/artifacts/**)`, `Edit(/absolute/project/path/.claude/artifacts/**)`). Create the file with `{"permissions":{"allow":[...]}}` if it doesn't exist. One-time setup — user approves once, then all artifact file ops are auto-approved.
5. Detect project language/framework → cache in `.claude/project-profile.json`
6. Generate slug + timestamp → create `.claude/artifacts/{date}/{slug}-{HHMM}/` → init meta.json + update index.json (with `path` field)

The artifact directory must exist before any research or analysis begins — all outputs are saved there, and late creation risks losing work.

### Step 2: Research — skip if `--direct` / `--no-research` / `--from`

1. Read `templates/research.md` for output structure
2. Check existing artifacts for same-module research (index.json) → reuse as baseline if recent (<30 days)
3. Spawn Explore agents (haiku) — minimum 1, up to scale table maximum. PM decides count based on codebase complexity within the scale limit.
4. Synthesize research.md (sonnet) with discovery IDs: H1, H2, M1, M2, L1…
5. Verify: Glob/Grep referenced files exist → fix stale references
6. Save to artifact dir → copy to project root
7. Update meta.json checkpoint: `"step": "research_done"`

### Step 3: Plan — skip if `--direct` / `--from plan.md`

1. Read `templates/plan.md` for output structure
2. Write plan (sonnet): [N-M] task IDs + [REF:Hx,Mx] traceability tags
3. Save to artifact dir → copy to project root
4. Traceability: all HIGH refs covered? No phantom refs?
5. Review (haiku × `--review`): assign from roles R1–R5 below. **Default `--review 1` uses R5.** Score: PASS(≥7) / NEEDS_REVISION(5–6) / FAIL(≤4)
   - R1: **Completeness** — are all HIGH findings addressed? Any missing tasks?
   - R2: **Feasibility** — are tasks technically achievable? Dependencies correct?
   - R3: **Risk** — are risks identified? Edge cases covered?
   - R4: **TDD/SOLID** — are test tasks before implementation? SOLID violations?
   - R5: **Goal Alignment** (default) — goal matches request? Research interpretation correct? Referenced files/classes exist? No irrelevant work?
6. Auto-fix on revision → re-save → re-copy
7. Update checkpoint: `"step": "plan_done"`

### Step 4: Checkpoint — skip if `--direct`

Display scale summary before asking:
- Total Phases / Total Tasks
- Estimated model allocation (e.g., "implement: sonnet ×12, review: opus ×12, QA: sonnet ×3")
- Exec strategy: subagent or team

AskUserQuestion (single_select): "Execute" / "Revise then execute" / "Cancel"
Cancel → meta.json status "cancelled", exit. Revise → collect feedback → modify plan → re-checkpoint.

### Step 5: Git Branch

Suggest `feature/{slug}` branch. Skip if already on feature branch or user declines.

### Step 6: Checklist → Tasks

- Parse `- [ ]` items → TaskCreate with `[N-M]` ID prefix
- `--direct` mode: PM analyzes request, designs tasks directly, writes minimal plan.md
- Set Phase dependencies: Phase N first task blocked by Phase N-1 last task

### Step 7: Agent Setup — Read prompts at spawn time

This step is always executed regardless of execution mode. Prompts contain domain knowledge (design principles, checklists, review criteria) that agents need to produce quality output.

1. **Read ALL required prompt files in parallel** (single message, multiple Read calls):
   - **code**: Read `prompts/implementer.md` + `prompts/code-reviewer.md` + `prompts/qa-inspector.md` + `checklists/general.md` + `checklists/{detected-lang}.md`
   - **docs**: Read `prompts/implementer.md` + `prompts/doc-reviewer.md`
   - **analysis**: Read `prompts/analyst.md` only
   - **infra/design**: as mapped above
   - **Checklist fallback**: If `checklists/{detected-lang}.md` does not exist, use `checklists/general.md` only. Log the fallback in meta.json.

2. **Reuse project rules**: `.claude/forge-rules.md` was already loaded in Step 1 — use the cached content directly.

3. **Placeholder substitution**: When spawning agents (Step 8), replace these placeholders in prompts:
   - `{PROJECT_RULES}` → contents of `.claude/forge-rules.md`
   - `{LANG_CHECKLIST}` → contents of `checklists/{detected-lang}.md`
   - `{GENERAL_CHECKLIST}` → contents of `checklists/general.md`

4. **Store in context**: Keep loaded prompts available for Step 8 agent spawning.

**team mode** (medium/large — preferred):
1. `TeamCreate` with slug as team name
2. Spawn 2 persistent agents via `Agent` tool with `team_name` parameter (in parallel):
   - **implementer** (`general-purpose`): code implementation, using substituted implementer.md prompt
   - **code-reviewer** (`general-purpose`, model: **opus**): code quality review, using substituted code-reviewer.md prompt
3. **qa-inspector** (`general-purpose`): spawned **on-demand at Phase boundaries only** (Step 8 item 6), using substituted qa-inspector.md prompt. Shutdown after QA pass — saves cost on phases with few tasks.
4. PM communicates with agents via `SendMessage`, agents report back via `SendMessage`

**subagent mode** (small only):
1. No TeamCreate — agents spawned per-task via `Agent` tool (no team_name)
2. Prompts loaded here, concatenated with substituted placeholders, passed to each Agent call
3. Each agent dies after returning result — PM manages all state

Note: "subagent mode" still requires this step — prompts are read here regardless of mode. Team mode spawns implementer + code-reviewer here (not deferred to Step 8). qa-inspector is spawned on-demand at Phase boundaries.

### Step 8: PM Execution Loop

**TDD (code type):** Test tasks execute before implementation tasks because writing tests first forces clearer interface design and catches regressions immediately. The plan must include test tasks. Skip only with explicit justification recorded in plan.md (e.g., runtime unavailable on current platform). OOP/SOLID compliance is mandatory — violations cause REJECT verdict because they create maintenance burden.

```
FOR EACH task (unblocked, ID ascending):
  1. Implementer: assign task (Agent call or SendMessage)
  2. GATE (6-item self-check — ALL must be OK/N/A, reject if any missing):
     a. Circular/self reference: no self-assignment or circular data flow?
     b. Init order: no event-triggered overwrites during construction/initialization?
     c. Null/empty safety: safe parsing used (e.g., TryParse in C#, try/except in Python, parseInt with fallback in JS)?
     d. Save→Load roundtrip: saved value restores identically?
     e. Event timing: no premature event/callback fire before values loaded?
     f. Build+Test: build success + test pass (or explicit skip reason)?
  3. PM structural check: plan coverage, file completeness, TDD compliance, SOLID compliance
  4. Reviewer: code-reviewer / doc-reviewer / skip (analysis)
  5. Verdict: PASS → complete | NEEDS_REVISION → loop | REJECT → AskUserQuestion: retry / skip (with justification logged in meta.json)
  6. Phase end → QA → git commit "feat({slug}): complete phase {N}"
  7. Phase boundary report (AskUserQuestion: continue / revise / stop):
     - Changed files list (with line count delta)
     - Review results summary (PASS/NEEDS_REVISION/REJECT counts)
     - Cumulative progress: completed tasks / total tasks
  8. meta.json real-time update
```

**Context efficiency:** Pass diff + surrounding context (not full file), current phase section (not full plan), task result summary (not full output) to agents.
**Limits:** minor revisions ≤5, major ≤3, reject ≤2, QA retries ≤2 → AskUserQuestion escalation.
**Timeouts (team):** implement 10m, review 5m, QA 5m → respawn 1× → escalation.
**Safety:** DB/config/security/API/dependency changes → user confirmation before execution.
**Parallel:** Independent same-Phase tasks run simultaneously (subagent: parallel Agent calls). Tasks modifying the same file(s) run sequentially to avoid conflicts.
**Adaptive:** 3+ revisions → suggest model upgrade. Frequent subagent revisions → suggest team switch.

### Step 9: Finalize

1. **Execution log verification**: Check meta.json `steps_completed` array. For the current request type, verify all MUST steps from the "Required Steps by Type" table were recorded. If any MUST step is missing, report the gap to the user via AskUserQuestion — don't mark as completed until resolved.
2. Shutdown agents (team: shutdown → 30s → TeamDelete)
3. Update plan.md: `- [ ]` → `- [x]` → re-copy to root (skip if no plan)
4. meta.json: status "completed", final stats verification
5. Read `templates/report.md` → generate final report → save to artifact dir + project root
6. Suggest PR creation (code/infra only)
7. Optional feedback → `.claude/artifacts/{date}/{slug}-{HHMM}/feedback.md`
8. Learning: if any new pattern detected in revisions → append to `.claude/forge-rules.md` in project root. Capture immediately on first detection — waiting for repetitions risks losing valuable insights.

**meta.json `steps_completed` format**: Array of step names completed during this run. Example: `["init", "research", "plan", "checkpoint", "git_branch", "tasks", "agent_setup", "execute", "finalize"]`. Each step adds itself to this array when completed. Step 9 verifies completeness.

**Every forge run produces at minimum**: meta.json + research.md + report.md in artifact dir. Without these, there's no record of what was done or why.

## Resume (`--resume slug`)

Read meta.json → verify "in_progress" → resume from checkpoint step. Restore exec_strategy.

## Init (`--init`)

New project: ask tech stack → scaffold structure. Existing: analyze → generate `project-profile.json`.

## Task ID System

Format: `N-M` (Phase-Task). Supplementary: `N-Ma`, `N-Mb`. Used in TaskCreate subject, progress output, meta.json checkpoint, agent communication.

## Artifact Structure

```
.claude/artifacts/{date}/{slug}-{HHMM}/ → research.md, plan.md, meta.json, feedback.md(opt)
```
- Date: `YYYY-MM-DD` (creation date)
- Slug: a-z0-9 + hyphens, max 30 chars
- HHMM: creation time in 24h format (e.g., `1430`)
- index.json: stores `slug`, `path` (`{date}/{slug}-{HHMM}`), `created` (`YYYY-MM-DDTHH:MM`)
- Artifact = master, root = one-way copy.
