# claw-code-evolution

Claw Code architecture analysis and evolution guide for AI Agents.

## What is this?

A deep analysis of [Claw Code](https://github.com/UltraWorkers/claw-code) (by UltraWorkers) — a Rust reimplementation of Claude Code — extracting 50 core design lessons for AI Agent architecture evolution.

Covers: system prompts, permission models, session compression, task subsystems, MCP lifecycle, hook interception, error recovery, green contracts, CLI architecture, rendering, telemetry, API abstraction, and more.

## Quick Install

```bash
curl -sL "https://github.com/1q7-i08mc07djv-dotcom/claw-code-evolution/raw/master/claw-code-evolution.skill" -o /tmp/claw-code-evolution.skill && clawhub install /tmp/claw-code-evolution.skill --dir ~/.openclaw/skills
```

> Requires [ClawHub CLI](https://clawhub.com): `npm i -g clawhub`

## Structure

```
claw-code-evolution/
├── SKILL.md              ← Entry point (trigger + quick nav)
└── references/
    └── 50-lessons.md     ← Full 50 lessons content
```

## Key Lessons Already Adopted

| Design | Lesson | Status |
|--------|--------|--------|
| Git context discovery | #1 | Adopted in AGENTS.md |
| Dangerous operation disclosure | #2 | Adopted in AGENTS.md |
| Workspace protection | #2 | Adopted in AGENTS.md |
| Git operation tiers | #2 | Adopted in AGENTS.md |
| Session compression merge | #3 | Design reference |

## Lessons by Category

| Category | Count | Highlights |
|----------|-------|------------|
| Core Systems | 8 | 5-level permissions, compression merge, hook interception |
| Runtime | 7 | ConversationRuntime pipeline, MCP lifecycle |
| Toolchain | 8 | FileOps safety, worker state machine |
| Infrastructure | 8 | Config discovery chain, sandbox isolation, OAuth |
| Ecosystem | 10 | CLI, Plugin, Telemetry, LSP, Skill |
| API Layer | 8 | Multi-Provider, error classification |

## Key Insight

Claw Code is highly engineered but **tool-oriented** — "You are an interactive agent that helps users with software engineering tasks."

Your Agent's edge: **soul** (SOUL.md), **proactive heartbeat**, **group chat awareness**, **skill hot-swap**.

## License

GPL v3 + Special Addition (same as SCL Launcher). See [LICENSE](LICENSE).
