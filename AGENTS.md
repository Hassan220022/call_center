# AGENTS.md

This file defines how sub-agents work in this repository.

## Non-Negotiable Product Rules

- All customer-facing prompts and responses must be in Egyptian Ammiya.
- Tool access must be enforced in code by domain-specific allowlists.
- Financial state changes always go to humans.
- False-promise claims and high-risk manipulation states trigger human escalation.
- Research notes are inputs, not source of truth.

## Source Of Truth

Use this precedence order:

1. `PROMPT.md`
2. assigned task file in `tasks/`
3. `AGENTS.md`
4. `CLAUDE.md`
5. `.claude/agents/research_notes/*.md`

## Before You Start

Every sub-agent must do these steps:

1. Read `PROMPT.md`.
2. Read the assigned task file.
3. Check `.claude/agents/research_notes/` for directly relevant notes.
4. Confirm task dependencies are satisfied.
5. Stay within the task’s ownership boundary unless the task file says otherwise.

## Swarm Rules

The project is designed for parallel work.

Prefer ownership by package boundary:

- `src/egyptian_voice_agent/config/`
- `src/egyptian_voice_agent/prompts/`
- `src/egyptian_voice_agent/agents/`
- `src/egyptian_voice_agent/security/`
- `src/egyptian_voice_agent/voice/`
- `src/egyptian_voice_agent/tools/`
- `src/egyptian_voice_agent/middleware/`
- `src/egyptian_voice_agent/pipeline/`
- `src/egyptian_voice_agent/telephony/`
- `tests/`
- `scripts/`

Do not rewrite architecture decisions inside implementation tasks. If the assigned task conflicts with `PROMPT.md`, stop and report the conflict.

## Bootstrap Safety

This repository starts from a planning state. Early tasks will create the codebase itself.

Until Phase 1 is completed:

- missing `src/`, `tests/`, `pyproject.toml`, and provider code are expected
- failing commands because the bootstrap does not exist yet are not task failure by themselves
- those gaps must be documented in the task context file

## Task File Workflow

Every implementation task must maintain its task file.

Task files live here:

- `tasks/phase-1-infrastructure/`
- `tasks/phase-2-prompts/`
- `tasks/phase-3-agents/`
- `tasks/phase-4-security/`
- `tasks/phase-5-voice/`
- `tasks/phase-6-tools/`
- `tasks/phase-7-middleware/`
- `tasks/phase-8-pipeline/`
- `tasks/phase-9-tests/`
- `tasks/phase-10-deployment/`

## Required Completion Steps

Before returning success, every sub-agent must:

1. update the task file status
2. check or update the acceptance criteria in that task file
3. create or update `.claude/context/{task_id}.md`
4. record what was implemented and what was verified

## Required Context File

Context files live at:

- `.claude/context/P1.1.md`
- `.claude/context/P2.1.md`
- and so on

Use this template:

```md
# Task P1.1: Example Task Name

## Status
COMPLETED

## Summary
- short summary of what changed

## Files
- `path/to/file`

## Verification
- `command run` -> Passed / Failed / Blocked

## Notes
- blockers, follow-ups, or caveats
```

## Required Task File Format

The first metadata block in every task file should stay machine-readable and easy to scan:

```md
# Task P1.1: Example Task Name

- **Phase:** 1 - Infrastructure
- **Status:** Not Started
- **Priority:** P0
- **Owner Role:** architect
- **Depends On:** None
- **Parallel Safe With:** None
- **Target Areas:** `path/one`, `path/two`
```

## Validation Commands

Use the narrowest useful validation first.

Examples once the bootstrap exists:

```bash
uv run ruff check src tests
uv run mypy src
uv run pytest
```

If those commands cannot run yet because the repo has not reached the relevant phase, record that explicitly in the context file instead of pretending they passed.

## Research Notes Policy

Check research notes before starting research-heavy tasks.

Create or update a research note when:

- a vendor integration required non-trivial investigation
- a legal/compliance issue was clarified
- a benchmark produced a decision that other tasks depend on

Research notes should capture:

- what is verified
- what is still provisional
- which official or primary sources were used
- what decision the finding affects

## Language And Safety Rules

Do not let implementation drift away from these constraints:

- customer-facing strings stay in Egyptian Ammiya
- prompts stay concise and phone-call oriented
- domain agents cannot call tools outside their allowlist
- risky financial, legal, or unverifiable requests escalate
- security state is tracked outside LLM context

## Completion Report Format

Every sub-agent should return a short report like this:

```md
## Task Completion Report

**Task ID:** P1.1
**Task Name:** Example Task Name

### Documentation Updated
- [x] Task file updated
- [x] Context file updated
- [x] Acceptance criteria reviewed

### Files Modified
- `path/to/file`

### Verification
- `command` -> Passed
- `command` -> Blocked
```

## Failure Conditions

A sub-agent task is not complete if any of these are true:

- the task file still says `Not Started` or `In Progress`
- the context file was not created or updated
- acceptance criteria were ignored
- the agent claimed checks passed without actually running them
- the agent changed architecture decisions outside the assigned task
