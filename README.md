# Forge — Autonomous Development Skill

Forge is a context-engineered autonomous development pipeline for Claude Code.
It runs on the [Vela](https://github.com/EcoKG/vela) skill execution framework.

## Features

- **17+ Specialized Agents**: researcher, planner, implementer, code-reviewer, QA, VPM, debugger, and more
- **Pipeline Modes**: standard, quick, trivial, analyze, debug, ralph
- **7-Layer Quality System**: deep work → self-check → peer review → QA → VPM → goal-backward → gate guard
- **Code Enforcement**: 9 gates prevent pipeline violations

## Installation

Forge is included as a default skill when you install Vela:

```bash
curl -fsSL https://raw.githubusercontent.com/EcoKG/vela/main/setup.sh | bash
```

Or install manually into an existing Vela installation:

```bash
git clone https://github.com/EcoKG/forge.git ~/.claude/skills/vela/skills/forge-dev
```

## Usage

```
/forge "implement JWT authentication"
/forge --quick "fix the login bug"
/forge --analyze "security audit"
/forge --debug "investigate memory leak"
```

## Requires

- [Vela](https://github.com/EcoKG/vela) framework installed
- Claude Code (Claude Max/Pro subscription)
- Node.js 18+

## License

MIT
