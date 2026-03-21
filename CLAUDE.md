# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## CRITICAL: SUB-AGENT COMPLIANCE REQUIREMENTS

**If you are a sub-agent, you MUST read and follow `AGENTS.md` completely.**

Before returning success, you MUST:

1. Update task file status in `tasks/`
2. Create/update context file in `.claude/context/{task_id}.md`
3. Check off acceptance criteria in task file

**RETURNING WITHOUT THESE UPDATES = FAILED TASK**

See `AGENTS.md` for exact commands and templates.

---

## Project Overview

**Egyptian AI Voice Agent** is a production-grade, real-time AI voice agent for Egyptian call centers. It replaces/augments human agents, speaks native Egyptian Arabic (Ammiya), maintains <800ms end-to-end latency, and uses a **multi-sub-agent architecture** where specialized agents handle different domains (orders, bookings, complaints, billing) orchestrated by a supervisor agent.

**Status:** Active development — building from scratch.

## Project Layout

```
call_center/
├── CLAUDE.md              # This file - guidance for Claude Code
├── AGENTS.md              # Agent usage guidelines and requirements
├── PROMPT.md              # The full implementation plan / PRD
├── tasks/                 # Individual task files with requirements & status
│   ├── phase-1-infrastructure/
│   ├── phase-2-prompts/
│   ├── phase-3-agents/
│   ├── phase-4-security/
│   ├── phase-5-voice/
│   ├── phase-6-tools/
│   ├── phase-7-middleware/
│   ├── phase-8-pipeline/
│   ├── phase-9-tests/
│   └── phase-10-deployment/
├── .claude/
│   ├── agents/
│   │   └── research_notes/    # Research documentation (prevents duplicate research)
│   │       └── {topic}.md
│   ├── context/           # Task implementation documentation (auto-generated)
│   │   └── {task_id}.md
│   └── skills/            # Development patterns and guidelines
└── egyptian-voice-agent/  # The actual application code
    ├── pyproject.toml
    ├── .env.example
    ├── Dockerfile
    ├── docker-compose.yml
    ├── src/
    │   ├── server.py
    │   ├── pipeline.py
    │   ├── config/
    │   ├── prompts/
    │   ├── agents/
    │   ├── tools/
    │   ├── security/
    │   ├── voice/
    │   ├── middleware/
    │   └── monitoring/
    ├── tests/
    │   └── red_team/
    └── scripts/
```

## Package Manager: uv Only

**CRITICAL:** Use `uv` exclusively for all Python package operations.

```bash
# Install dependencies
uv sync

# Add packages
uv add <package>
uv add --dev <dev-package>

# Run application
uv run uvicorn src.server:app --host 0.0.0.0 --port 8000

# Run tests
uv run pytest

# Run linting
uv run ruff check src/ tests/

# Run formatting
uv run ruff format src/ tests/

# Type checking
uv run mypy src/
```

**Never use:** pip install, pip freeze, or requirements.txt in this project.

## Tech Stack

| Layer          | Technology                                      |
| -------------- | ----------------------------------------------- |
| Runtime        | Python 3.12+                                    |
| Framework      | FastAPI + Pipecat (voice agent orchestration)   |
| STT            | Deepgram Nova-3 Arabic (ar-EG dialect)          |
| LLM (Primary)  | Anthropic Claude Sonnet 4 (sub-agent responses) |
| LLM (Fast)     | OpenAI GPT-4o-mini (supervisor classification)  |
| TTS (Primary)  | Cartesia Sonic-3 (40ms TTFA)                    |
| TTS (Fallback) | ElevenLabs Flash v2.5                           |
| Telephony      | Twilio Elastic SIP Trunk via WebSocket          |
| VAD            | Silero VAD v5 + Pipecat SmartTurn               |
| Monitoring     | Prometheus metrics + structured JSON logging    |
| Testing        | pytest + pytest-asyncio + red-team test suite   |

## Architecture: Multi-Sub-Agent Pattern

The system uses a **Supervisor -> Sub-Agent** routing pattern:

```
Caller speaks
     |
     v
STT (Deepgram Nova-3 ar-EG)
     |
     v
Pre-LLM Security Layer (injection_detector, manipulation_detector)
     |
     v
Supervisor Agent (GPT-4o-mini, intent classification only, <150ms)
     |
     v
Sub-Agent (Claude Sonnet 4, domain-specific response)
  - Orders Agent    (tools: check_order, track_delivery)
  - Booking Agent   (tools: book/reschedule/cancel appointment)
  - Complaints Agent (tools: log_complaint, transfer_to_human)
  - Billing Agent   (tools: get_invoice, check_payment)
  - Fallback Agent  (tools: search_faq)
     |
     v
Post-LLM Output Filter (blocks leaked secrets/policies)
     |
     v
TTS (Cartesia Sonic-3, Egyptian Cairene voice)
     |
     v
Twilio Audio Out -> Caller
```

### Why Sub-Agents Instead of One Giant Agent?

1. **Smaller prompts = faster TTFT** — each sub-agent has a focused ~500-token prompt
2. **Tighter guardrails** — orders agent literally cannot access billing tools (least privilege)
3. **Better quality** — specialist prompts generate better responses than generalist
4. **Easier to maintain** — update booking flow without risking orders or complaints
5. **Measurable** — track accuracy/satisfaction per domain independently

## Key Design Decisions (DO NOT DEVIATE)

1. **ALL prompts MUST be in Egyptian Ammiya.** Zero Fusha. If the LLM generates "saaqoom" or "hal yumkinuni", the prompt has failed.

2. **Supervisor uses a FAST model** (GPT-4o-mini or Claude Haiku) for classification only. Sub-agents use Claude Sonnet for response quality.

3. **Each sub-agent has a WHITELIST of tools.** The orders agent literally cannot call billing tools. This is enforced in code, not just in the prompt.

4. **The manipulation detector runs OUTSIDE the LLM context.** It's pure Python keyword matching on the STT transcript. Even if the prompt is bypassed, the code-level tracker forces a transfer.

5. **False promise claims trigger IMMEDIATE human transfer.** The AI cannot verify what a previous agent said. No exceptions.

6. **Financial actions ALWAYS go to humans.** The AI never processes refunds, discounts, credits, or price changes. These tools don't even exist in the agent's tool set.

7. **The voice clone is NON-NEGOTIABLE.** A generic Arabic TTS voice will sound like an Al Jazeera newsreader. Must clone from a real Cairene Egyptian speaker.

8. **All responses are 1-3 sentences.** This is a voice call, not a chatbot.

9. **The golden rule: "Friendly and professional, but not weak."** Empathy does not equal surrender.

10. **When in doubt, transfer to human.** Better to transfer 100 extra calls than lose money on 1 manipulation.

## Latency Targets

| Component  | Target |
| ---------- | ------ |
| End-to-end | <800ms |
| STT        | <100ms |
| LLM TTFT   | <250ms |
| TTS TTFA   | <90ms  |
| Supervisor | <150ms |

## Working with This Repository

### Reading the Implementation Plan

The `PROMPT.md` file is the comprehensive source of truth. It contains:

- Complete project structure (every file described)
- Multi-sub-agent architecture diagram
- File-by-file implementation instructions
- Security layer design
- Voice service configuration
- 10-phase implementation order

### Task Workflow

1. Read recent progress in task files
2. Find the highest priority incomplete task
3. Implement exactly ONE task
4. Run checks: `uv run ruff check src/ tests/ && uv run pytest`
5. Log progress by updating the task file
6. **Generate task context documentation** (see below)
7. Commit changes

### Checking Task Status

**To quickly check the status of any task, read the first 9 lines of the task `.md` file inside the `tasks/` folder.** These lines contain the essential metadata including the current status.

### Task Completion Documentation (MANDATORY)

**CRITICAL FOR ALL AGENTS (INCLUDING SUB-AGENTS):**

**When a task is completed, you MUST update THREE locations:**

#### 1. Update Task File Status in `tasks/`

Find the task file and update the **Status** field:

**Before:**

```markdown
- **Status:** Not Started
```

**After (completed successfully):**

```markdown
- **Status:** Completed
```

Also update the acceptance criteria checkboxes:

```markdown
## Acceptance Criteria

- [x] Pydantic BaseSettings configured with all env vars
- [x] Constants file with all latency targets
- [x] .env.example documented
```

#### 2. Create/Update Task Context File

**Path Pattern:** `.claude/context/{task_id}.md`

Create/update the context file with:

1. **Header:** `# Task {task_id}: {Task Name}`
2. **Status:** One of `COMPLETED`, `NEEDS FIXES`, or `IN PROGRESS`
3. **Implementation Summary:** Brief description of what was implemented
4. **Files Created/Modified:** List each file with description
5. **Key Implementation Details:** Architectural decisions, patterns, dependencies
6. **Verification Results:** Output of lint, typecheck, test runs
7. **Known Issues / TODOs:** Any remaining issues
8. **Correction Log:** If implementation was incorrect and fixed later

#### 3. Reference Context When Confused

**Subagents MUST check `.claude/context/{task_id}.md` before:**

- Re-implementing a task that was already done
- Debugging issues in previously completed features
- Understanding how a feature was implemented

---

## Research Notes System

**Purpose:** Prevent duplicate research and provide fast understanding for all agents.

**Location:** `.claude/agents/research_notes/`

**Naming Convention:** `{topic}.md`

### When to Create Research Notes

Create a research note when:

- Investigating a new technology, pattern, or integration approach
- Completing deep-dive research on a complex topic (Pipecat, Deepgram, Cartesia, etc.)
- Documenting findings that will be useful for future tasks
- Solving a problem that required significant research effort

### Research Note Structure

```markdown
# Research Note: {Topic}

## Summary

Brief 2-3 sentence overview of the research findings.

## Key Findings

- Bullet points of the most important discoveries
- Patterns, best practices, or gotchas found

## Implementation Details

Code examples, configuration snippets, or step-by-step guides.

## References

- Links to documentation, articles, or other resources used

## Related Tasks

- Task IDs that used or will use this research
```

### Using Research Notes

**Before starting research on any topic:**

1. Check `.claude/agents/research_notes/` for existing notes
2. Use `glob` or `grep` to search for relevant topics
3. If a note exists, read it first to avoid duplicate work

---

## SUB-AGENT MANDATORY REQUIREMENTS

> **CRITICAL**: Sub-agents that do not update task files are considered **FAILED**, even if the implementation is correct.

When you are a sub-agent (spawned via `task` tool), you **MUST** perform these updates before reporting completion:

### Required Updates Checklist

```
1. Check research notes FIRST in .claude/agents/research_notes/
2. Update task file in tasks/phase-{N}-{name}/{task_id}-*.md
   - Status -> Completed
   - Check off acceptance criteria
3. Create/Update context file in .claude/context/{task_id}.md
4. Create research note if significant research was done
```

### VALIDATION BEFORE RETURNING

```bash
# Validate task file was updated (replace TASK_ID)
TASK_ID="P1.1"
PHASE_DIR=$(ls -d tasks/phase-*/ | head -1)
grep -l "Status.*Completed" ${PHASE_DIR}${TASK_ID}-*.md && echo "Task file OK" || echo "TASK FILE NOT UPDATED!"

# Validate context file exists
ls .claude/context/${TASK_ID}.md && echo "Context file OK" || echo "CONTEXT FILE MISSING!"
```

### REQUIRED OUTPUT FORMAT

Your final response MUST include:

```
## Task Completion Report

**Task ID:** P{X}.{Y}

### Documentation Updated:
- [x] Task file: `tasks/phase-{N}-*/...` -> Status: Completed
- [x] Context file: `.claude/context/P{X}.{Y}.md` -> Created

### Files Modified:
- `path/to/file.py`

### Verification:
- `uv run ruff check` -> Passed
- `uv run pytest` -> Passed
```

**NO COMPLETION REPORT = TASK FAILED**

---

### Git Workflow

- Create one commit per task with clear message
- Do NOT run `git init` or change remotes
- Do NOT push unless explicitly requested

## Security Requirements

- Pre-LLM injection detection (regex + classifier on STT output)
- Tool argument validation + rate limiting per session
- Post-LLM output filtering before TTS (block leaked secrets/policies)
- CRM data sanitization (prevent indirect injection via poisoned data)
- Per-call security state machine tracking injection + manipulation attempts
- Financial actions ALWAYS require human handoff
- False promise claims trigger IMMEDIATE human transfer

## Egyptian Arabic (Ammiya) Requirements

- ALL agent prompts and responses in Egyptian Ammiya dialect
- Zero Modern Standard Arabic (Fusha) — no saaqoom, hal yumkinuni, madha, etc.
- Egyptian honorifics: ya fandem, ya basha, ya hanem, hadretak
- Egyptian expressions: in sha allah, taht amrak, khalas, ma feesh moshkela
- Vowel stretching for emotion: ya salaaaam!, tab3aaaa, ayoooooh
- Natural fillers: ya3ni..., ummm..., boss ya fandem...
- Cairene phonetic markers: qaf->hamza, geem->g, dhal->dal, tha->ta
- All responses 1-3 sentences max (voice call, not chatbot)

## Required Agents

**CRITICAL:** Use the appropriate specialized agent for each task.

### Development Agents

| Agent                              | When to Use                                                                 |
| ---------------------------------- | --------------------------------------------------------------------------- |
| `project-starter:orchestrator`     | Complex multi-step tasks, 2+ modules, open-ended requests                   |
| `project-starter:code-reviewer`    | After writing or modifying code, before commits                             |
| `project-starter:debugger`         | Any error, exception, failing test, or unexpected behavior                  |
| `project-starter:test-architect`   | Adding new features, fixing bugs, improving test coverage                   |
| `project-starter:security-auditor` | Security layer code, injection detection, tool guardrails, output filtering |
| `project-starter:refactorer`       | Code quality improvements, reducing technical debt                          |

### Workflow Agents

| Agent                                        | When to Use                                                                    |
| -------------------------------------------- | ------------------------------------------------------------------------------ |
| `superpowers:brainstorming`                  | **MUST use before** any creative work - creating features, building components |
| `superpowers:systematic-debugging`           | Before proposing fixes for any bug or test failure                             |
| `superpowers:test-driven-development`        | Before writing implementation code for features or bugfixes                    |
| `superpowers:writing-plans`                  | When you have specs/requirements for a multi-step task                         |
| `superpowers:verification-before-completion` | Before claiming work is complete or creating PRs                               |

### Agent Usage Rules

1. **Check research notes FIRST** - Always check `.claude/agents/research_notes/` before researching
2. **Check for applicable agents** before starting any task
3. **Complex tasks = orchestrator** - When touching multiple modules
4. **After code changes = code-reviewer** - Before committing
5. **Errors/failures = debugger** - Before attempting fixes
6. **Security-sensitive = security-auditor** - Injection detection, guardrails, output filtering
7. **After significant research = Create research note** - Document findings for future agents
