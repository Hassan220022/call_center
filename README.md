# Egyptian AI Voice Agent

This repository is the bootstrap plan for an Egyptian Arabic call-center voice agent.

Current status on March 22, 2026:

- The repository is now doc-first and task-driven.
- The application code has not been built yet.
- The canonical implementation spec lives in `PROMPT.md`.
- The executable swarm plan lives in `tasks/`.
- Research notes under `.claude/agents/research_notes/` are inputs, not source of truth.

## Read First

1. `PROMPT.md` for the target architecture, locked decisions, and implementation rules.
2. `tasks/README.md` for execution waves and the task index.
3. `AGENTS.md` for sub-agent behavior, ownership, and completion rules.
4. `CLAUDE.md` for repo workflow and source-of-truth precedence.
5. `.claude/agents/research_notes/` for supporting research and open questions.

## Project Goal

Build a production-grade voice agent for Egyptian call centers that:

- speaks Egyptian Ammiya, not Fusha
- answers briefly enough for phone calls
- uses code-level tool isolation
- transfers risky or unverifiable situations to humans
- is measurable, testable, and deployable in a legally reviewed Egyptian setup

## Current Repo State

What exists now:

- top-level operating docs
- research notes
- a complete phase/task plan for sub-agents

What does not exist yet:

- `src/`
- `tests/`
- provider integrations
- a runnable FastAPI or Pipecat service

Do not claim the system is implemented until the tasks are actually completed.

## Canonical Decisions

These are the current architecture decisions for the first implementation pass.

| Area | Decision | Status |
| --- | --- | --- |
| Runtime | Python 3.12+, `uv`, `ruff`, `mypy`, `pytest` | Locked |
| App shape | Root Python project with `src/egyptian_voice_agent/` | Locked |
| API layer | FastAPI session server | Locked |
| Voice orchestration | Pipecat flows and adapters | Locked |
| Language rule | All customer-facing prompts and responses in Egyptian Ammiya | Locked |
| Agent boundaries | Domain agents with code-level tool allowlists | Locked |
| STT | Deepgram `nova-3` with `ar-EG` as the default first integration | Locked |
| Router | Deterministic plus embedding router with LLM fallback | Locked |
| Security | Arabic normalization, injection scanning, manipulation state machine, output filtering | Locked |
| Telephony | Provider adapter layer; production provider requires legal and commercial validation | Locked |
| Dev telephony | Twilio Media Streams is the initial development transport | Provisional |
| Egypt production telephony | Telnyx or another Egypt-compliant provider after legal and commercial review | Needs legal review |
| Primary text LLM | OpenAI-compatible adapter is the default first implementation path | Provisional |
| Challenger text LLM | Gemini adapter is benchmarked before production lock | Needs benchmark |
| Primary TTS | Benchmark-driven selection between ElevenLabs PVC and Cartesia low-latency path | Needs benchmark |

## Architecture Summary

The target system is:

1. caller audio enters through a telephony transport adapter
2. VAD and turn logic segment speech
3. STT produces Egyptian-Arabic transcripts
4. Arabic normalization and pre-LLM security checks run outside model context
5. a router selects the active domain agent
6. the selected agent runs with its own prompt, short context window, and tool allowlist
7. tool calls pass through schema validation and policy guardrails
8. output filtering validates the final text
9. TTS renders Egyptian-friendly speech
10. metrics, traces, and audit events are emitted through the whole path

Financial changes, unverifiable “the other agent promised me” claims, and high-risk manipulation states always route to a human.

## Repository Map

Current authoritative docs:

- `PROMPT.md`: architecture and delivery spec
- `tasks/README.md`: swarm execution plan
- `AGENTS.md`: sub-agent operating contract
- `CLAUDE.md`: repo workflow for AI coding agents

Supporting docs:

- `.claude/agents/research_notes/README.md`: how to use the research notes
- `.claude/agents/research_notes/*.md`: provisional research summaries and open questions

Target project layout after Phase 1:

```text
call_center/
├── README.md
├── PROMPT.md
├── CLAUDE.md
├── AGENTS.md
├── pyproject.toml
├── .env.example
├── src/
│   └── egyptian_voice_agent/
│       ├── app/
│       ├── config/
│       ├── prompts/
│       ├── agents/
│       ├── security/
│       ├── voice/
│       ├── tools/
│       ├── middleware/
│       ├── pipeline/
│       ├── telephony/
│       └── monitoring/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── red_team/
├── scripts/
├── tasks/
└── .claude/
```

## Swarm Start Order

The swarm should not start coding from the old research notes. It should start from the task plan.

Recommended order:

1. `P1.1` architecture freeze and ADR capture
2. `P1.2` repository bootstrap foundation
3. `P2.1`, `P4.1`, and `P5.1` as the first parallel contract wave
4. `P2.2`, `P3.1`, `P4.2`, `P5.2`, and `P6.1` as the second parallel wave
5. pipeline integration only after stable interfaces, middleware, and tools exist

## Research Notes Policy

The research notes are intentionally no longer treated as “validated final choices.”

Use them as:

- vendor and architecture inputs
- benchmark ideas
- legal and operational reminders

Do not use them as:

- canonical implementation spec
- proof that a vendor decision is final
- substitute for legal review or benchmarking

## Immediate Next Step

The next agent should read `PROMPT.md`, then `tasks/README.md`, then start `tasks/phase-1-infrastructure/P1.1-architecture-freeze.md`.
