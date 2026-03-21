# AGENTS.md

This file provides guidance for all AI agents (main agents, sub-agents, and any spawned agents) working on the Egyptian AI Voice Agent project.

> **Key Constraint:** ALL voice agent prompts and responses MUST be in Egyptian Ammiya (colloquial Arabic).
> Zero Modern Standard Arabic (Fusha). If the LLM generates formal Arabic, the prompt has failed.
> **Design Rule:** Each sub-agent has a tool WHITELIST enforced in code. Orders agent cannot call billing tools. Period.

---

## STOP! READ THIS FIRST - MANDATORY FOR ALL SUB-AGENTS

**If you are a sub-agent spawned via the `task` tool, you MUST complete these steps BEFORE returning success:**

### QUICK COMPLIANCE CHECKLIST (Copy this into your final response)

```
## Task Completion Checklist
- [ ] Implementation complete
- [ ] Task file updated: tasks/phase-{N}-*/P{X}.{Y}-*.md (Status -> Completed)
- [ ] Context file created/updated: .claude/context/P{X}.{Y}.md
- [ ] Acceptance criteria checkboxes checked in task file
```

### EXACT COMMANDS TO RUN BEFORE COMPLETION

**Step 1: Update Task File Status**

```bash
# Find your task file (replace P1.1 with your task ID)
TASK_ID="P1.1"
TASK_FILE=$(find tasks/ -name "${TASK_ID}-*.md" 2>/dev/null | head -1)
echo "Task file: $TASK_FILE"

# Update status to Completed
sed -i '' 's/- \*\*Status:\*\*.*/- **Status:** Completed/' "$TASK_FILE"
```

**Step 2: Create/Update Context File**

```bash
# Create context file (replace with your task ID)
cat > .claude/context/P1.1.md << 'EOF'
# Task P1.1: [Task Name]

## Status: COMPLETED

## Implementation Summary
[Brief description of what was implemented]

## Files Created/Modified
- `path/to/file1.py` - Description
- `path/to/file2.py` - Description

## Verification
- [ ] `uv run ruff check` - Passed
- [ ] `uv run pytest` - Passed
EOF
```

### TASK IS NOT COMPLETE UNTIL:

1. Task file shows `- **Status:** Completed`
2. Context file exists at `.claude/context/{task_id}.md`
3. Acceptance criteria are checked off in task file

### DO NOT:

- Return "task complete" without updating files
- Skip the task file status update
- Forget to create the context file

---

## Research Notes System (CRITICAL)

**Purpose:** Prevent duplicate research and provide fast understanding for all agents.

**Location:** `.claude/agents/research_notes/`

**Naming Convention:** `{topic}.md`

### MANDATORY: Check Research Notes FIRST

**Before starting any research or investigation:**

1. **ALWAYS check** `.claude/agents/research_notes/` for existing notes
2. Use `glob` or `grep` to search: `glob pattern: ".claude/agents/research_notes/*.md"`
3. If a note exists on your topic, **READ IT FIRST** to avoid duplicate work
4. Use existing research as your starting point

### When to Create Research Notes

Create a research note when:

- Investigating Pipecat pipeline patterns or configuration
- Researching Deepgram STT Arabic dialect tuning
- Researching Cartesia/ElevenLabs TTS Egyptian voice configuration
- Investigating Twilio WebSocket media stream integration
- Researching Silero VAD tuning parameters
- Documenting Egyptian Arabic prompt engineering findings
- Solving a problem that required significant research effort

### After Completing Significant Research

1. Create a research note documenting your findings
2. Use descriptive topic names (e.g., `pipecat_pipeline_assembly`, `deepgram_arabic_config`, `cartesia_voice_cloning`)
3. Include practical code examples and gotchas
4. Link to related tasks that will benefit

---

## Required Agents

**CRITICAL:** Use the appropriate specialized agent for each task. These agents MUST be used when their domain applies.

### Development Agents

| Agent                              | When to Use                                                                 |
| ---------------------------------- | --------------------------------------------------------------------------- |
| `project-starter:orchestrator`     | Complex multi-step tasks, 2+ modules, open-ended requests                   |
| `project-starter:code-reviewer`    | After writing or modifying code, before commits                             |
| `project-starter:debugger`         | Any error, exception, failing test, or unexpected behavior                  |
| `project-starter:test-architect`   | Adding new features, fixing bugs, improving test coverage                   |
| `project-starter:security-auditor` | Security layer code, injection detection, tool guardrails, output filtering |
| `project-starter:refactorer`       | Code quality improvements, reducing technical debt                          |
| `project-starter:docs-writer`      | Creating README files, API documentation, architecture docs                 |

### Workflow Agents

| Agent                                        | When to Use                                                                    |
| -------------------------------------------- | ------------------------------------------------------------------------------ |
| `superpowers:brainstorming`                  | **MUST use before** any creative work - creating features, building components |
| `superpowers:systematic-debugging`           | Before proposing fixes for any bug or test failure                             |
| `superpowers:test-driven-development`        | Before writing implementation code for features or bugfixes                    |
| `superpowers:writing-plans`                  | When you have specs/requirements for a multi-step task                         |
| `superpowers:verification-before-completion` | Before claiming work is complete or creating PRs                               |
| `superpowers:requesting-code-review`         | After completing tasks or implementing major features                          |

### Exploration & Planning Agents

| Agent     | When to Use                                                                              |
| --------- | ---------------------------------------------------------------------------------------- |
| `Explore` | Finding files by patterns, searching code for keywords, understanding codebase structure |
| `Plan`    | Designing implementation strategy, step-by-step plans, architectural trade-offs          |

### Agent Usage Rules

1. **Check research notes FIRST** - Always check `.claude/agents/research_notes/` before researching
2. **Check for applicable agents** before starting any task
3. **Complex tasks = orchestrator** - When touching multiple modules
4. **After code changes = code-reviewer** - Before committing
5. **Errors/failures = debugger** - Before attempting fixes
6. **Security-sensitive = security-auditor** - Injection detection, guardrails, output filtering, session tracking
7. **After significant research = Create research note** - Document findings for future agents

---

## SUB-AGENT MANDATORY REQUIREMENTS

**THIS SECTION IS NON-NEGOTIABLE FOR ALL SUB-AGENTS**

> **CRITICAL**: Sub-agents that do not update task files are considered **FAILED**,
> even if the implementation is correct. The task is NOT complete until documentation is updated.

When you are a sub-agent (spawned via `task` tool), you **MUST** perform these updates before reporting completion:

### Required Updates Checklist

```
1. Check research notes FIRST in .claude/agents/research_notes/
   - Read any relevant existing research before starting

2. Update task file in tasks/phase-{N}-{name}/{task_id}-*.md
   - Status -> Completed
   - Check off acceptance criteria

3. Create/Update context file in .claude/context/{task_id}.md
   - Document what was implemented with file list

4. Create research note if significant research was done
   - Save to .claude/agents/research_notes/{topic}.md
```

### VALIDATION BEFORE RETURNING

Before you return success, run this validation:

```bash
# Validate task file was updated (replace TASK_ID)
TASK_ID="P1.1"
find tasks/ -name "${TASK_ID}-*.md" -exec grep -l "Status.*Completed" {} \; && echo "Task file OK" || echo "TASK FILE NOT UPDATED!"

# Validate context file exists
ls .claude/context/${TASK_ID}.md && echo "Context file OK" || echo "CONTEXT FILE MISSING!"
```

### How to Find Files

**Research Notes Location:**

- Always at `.claude/agents/research_notes/{topic}.md`
- Search with: `glob pattern: ".claude/agents/research_notes/*{keyword}*.md"`

**Task File Location:**

1. Extract phase number from task ID (e.g., P**1**.1 -> phase-1)
2. Map phase number to folder name:
   - Phase 1 -> `tasks/phase-1-infrastructure/`
   - Phase 2 -> `tasks/phase-2-prompts/`
   - Phase 3 -> `tasks/phase-3-agents/`
   - Phase 4 -> `tasks/phase-4-security/`
   - Phase 5 -> `tasks/phase-5-voice/`
   - Phase 6 -> `tasks/phase-6-tools/`
   - Phase 7 -> `tasks/phase-7-middleware/`
   - Phase 8 -> `tasks/phase-8-pipeline/`
   - Phase 9 -> `tasks/phase-9-tests/`
   - Phase 10 -> `tasks/phase-10-deployment/`
3. Find file matching `{task_id}-*.md` pattern

**Context File Location:**

- Always at `.claude/context/{task_id}.md`
- Create if it doesn't exist
- Update if it already exists

### Sub-Agent Completion Protocol

**FINAL CHECKLIST - DO NOT SKIP**

Before returning success to the parent agent, verify ALL items:

```markdown
1. Research notes checked (no duplicate research)
2. Implementation complete
3. Tests passing (if applicable)
4. Task file status updated in tasks/phase-X-\*/ -> "Completed"
5. Context file created/updated in .claude/context/
6. Acceptance criteria checkboxes updated
7. Research note created (if significant research done)
```

### REQUIRED OUTPUT FORMAT

Your final response MUST include this completion report:

```
## Task Completion Report

**Task ID:** P{X}.{Y}
**Task Name:** [Name]

### Documentation Updated:
- [x] Task file: `tasks/phase-{N}-*/P{X}.{Y}-*.md` -> Status: Completed
- [x] Context file: `.claude/context/P{X}.{Y}.md` -> Created/Updated
- [x] Acceptance criteria: All checked

### Files Modified:
- `path/to/file1.py`
- `path/to/file2.py`

### Verification:
- `uv run ruff check` -> Passed
- `uv run pytest` -> Passed
```

**FAILURE TO INCLUDE THIS REPORT = TASK NOT COMPLETE**
**FAILURE TO UPDATE DOCUMENTATION FILES = TASK NOT COMPLETE**

---

## Quick Reference

### Key Directories

| Directory                        | Purpose                               |
| -------------------------------- | ------------------------------------- |
| `.claude/agents/research_notes/` | Research documentation (CHECK FIRST!) |
| `.claude/context/`               | Task implementation documentation     |
| `.claude/skills/`                | Development patterns and guidelines   |
| `tasks/`                         | Task requirements and status          |
| `egyptian-voice-agent/src/`      | Application source code               |
| `egyptian-voice-agent/tests/`    | Test suite including red-team tests   |

### File Naming Patterns

| Type          | Pattern                    | Example                        |
| ------------- | -------------------------- | ------------------------------ |
| Research Note | `{topic}.md`               | `pipecat_pipeline_assembly.md` |
| Task Context  | `{task_id}.md`             | `P1.1.md`                      |
| Task File     | `{task_id}-{task-name}.md` | `P1.1-project-skeleton.md`     |

### Implementation Phases

| Phase | Name           | Description                                        |
| ----- | -------------- | -------------------------------------------------- |
| 1     | Infrastructure | Project skeleton, config, settings, constants      |
| 2     | Prompts        | Egyptian personality, security, business policy    |
| 3     | Agents         | Supervisor, base agent, all sub-agents, registry   |
| 4     | Security       | Injection detection, guardrails, output filter     |
| 5     | Voice          | STT, TTS, VAD, fallback TTS                        |
| 6     | Tools          | Tool definitions, handlers, CRM client, Twilio     |
| 7     | Middleware     | Manipulation detector, sentiment, auto-escalation  |
| 8     | Pipeline       | Pipecat pipeline assembly + FastAPI server         |
| 9     | Tests          | Unit tests, red-team suite, dialect quality checks |
| 10    | Deployment     | Dockerfile, docker-compose, scripts                |
