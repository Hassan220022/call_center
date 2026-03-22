# CLAUDE.md

This file gives repository-specific instructions to coding agents working in this project.

## Repo Reality

This is not a finished codebase.

As of March 22, 2026, this repository contains:

- the canonical architecture spec in `PROMPT.md`
- the sub-agent operating contract in `AGENTS.md`
- the execution plan in `tasks/`
- supporting research notes in `.claude/agents/research_notes/`

It does not yet contain the application package, tests, or deployment artifacts.

Do not claim `src/`, `tests/`, `pyproject.toml`, provider adapters, or the server already exist unless you created them in the current task.

## Source Of Truth Order

When docs disagree, use this order:

1. `PROMPT.md`
2. `tasks/`
3. `AGENTS.md`
4. `CLAUDE.md`
5. `.claude/agents/research_notes/*.md`

Research notes are supporting inputs, not final decisions.

## Working Model

The project is being built by a swarm of sub-agents. Work must be:

- task-driven
- contract-first
- adapter-first
- safe for parallel execution

Before implementing, read:

1. `PROMPT.md`
2. the assigned task file in `tasks/`
3. any directly relevant research note

## Current Architecture Position

The architecture is frozen at the doc level, with a few benchmark-gated choices still marked provisional.

Locked:

- FastAPI + Pipecat
- Egyptian Ammiya-only customer-facing language
- code-level tool allowlists per domain agent
- Deepgram `nova-3` with `ar-EG` as first STT integration
- hybrid routing strategy
- multi-layer security outside LLM context
- human escalation for risky financial or unverifiable cases

Provisional and benchmark-gated:

- final production telephony provider
- final production text LLM
- final production TTS default

## Package And Tooling Target

The target Python project layout is:

```text
src/egyptian_voice_agent/
tests/
scripts/
```

When Phase 1 creates the codebase, use:

- `uv` for environment and dependency management
- `ruff` for lint and formatting
- `mypy` for type checking
- `pytest` for tests

## Task Workflow

For implementation work:

1. Read the assigned task file.
2. Confirm dependencies are already completed or explicitly waived.
3. Read only the research notes directly relevant to the task.
4. Implement the task in the files owned by that task.
5. Run the narrowest useful verification first, then broader checks when available.
6. Update the task file status and acceptance criteria.
7. Create or update `.claude/context/{task_id}.md`.

If the repo bootstrap is incomplete and a command cannot run because Phase 1 has not built the project yet, say so clearly in the task context file.

## Parallel Work Rules

Prefer tasks that own disjoint areas:

- `config/`
- `prompts/`
- `agents/`
- `security/`
- `voice/`
- `tools/`
- `middleware/`
- `pipeline/`
- `telephony/`
- `tests/`
- `deployment/`

Do not silently change another task’s ownership boundary. If a task has to cross boundaries, update the task file or raise it explicitly.

## Prompt And Language Rules

Every customer-facing prompt and generated response must be in Egyptian Ammiya.

Never ship:

- Fusha-heavy phrasing
- long chatbot-style answers
- promises about refunds, credits, discounts, or policy exceptions
- “previous agent promised X” acknowledgments without escalation

Target spoken responses:

- 1 to 3 sentences
- natural phone-call rhythm
- respectful Egyptian honorifics
- concise enough for low-latency TTS

## Security Rules

The security model is not optional.

Always preserve these boundaries:

- Arabic normalization happens before security scanning.
- manipulation tracking happens outside the LLM context
- output filtering happens before TTS
- tool access is enforced in code, not only in prompts
- financial state changes require a human

## Telephony And Compliance Rules

Treat telephony and compliance as design constraints, not implementation details.

Hard requirements:

- recording must be consent-gated
- Egypt call-center/cloud-call-center operation requires legal review
- production provider choice is not final until commercial and legal validation are complete

Do not hard-code the stack to a single telephony vendor in shared interfaces.

## Task Completion

A task is not complete until:

1. the implementation is done
2. the task file is updated
3. `.claude/context/{task_id}.md` is updated
4. the acceptance criteria are checked or explicitly marked blocked

Use the exact workflow described in `AGENTS.md`.
