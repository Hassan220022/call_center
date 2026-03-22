# Task Plan

This directory is the executable plan for the sub-agent swarm.

## How To Use This Plan

1. Read `PROMPT.md`.
2. Start with the earliest incomplete dependency.
3. Work one task at a time unless the tasks are explicitly parallel-safe.
4. Update the task file status and acceptance criteria when done.
5. Create or update `.claude/context/{task_id}.md`.

## Execution Rules

- The swarm works contract-first.
- Provider-specific work starts only after interface tasks exist.
- Ownership follows package boundaries to reduce merge conflicts.
- Integration tasks come after contracts, adapters, and policy layers.

## Recommended Waves

Wave 1:

- `P1.1`

Wave 2:

- `P1.2`

Wave 3:

- `P2.1`
- `P4.1`
- `P5.1`

Wave 4:

- `P2.2`
- `P3.1`
- `P4.2`
- `P5.2`
- `P6.1`

Wave 5:

- `P3.2`
- `P6.2`
- `P7.1`

Wave 6:

- `P7.2`
- `P8.1`

Wave 7:

- `P8.2`
- `P9.1`
- `P9.2`

Wave 8:

- `P10.1`
- `P10.2`

## Task Index

### Phase 1

- `P1.1` Architecture Freeze And ADR Capture
- `P1.2` Repository Bootstrap Foundation

### Phase 2

- `P2.1` Prompt Contract And Base Persona
- `P2.2` Domain Prompt Packs And Prompt QA

### Phase 3

- `P3.1` Agent Core And Registry
- `P3.2` Router, Handoffs, And Context Strategy

### Phase 4

- `P4.1` Arabic Normalization And Injection Detection
- `P4.2` Manipulation State, Output Filtering, And Tool Safety

### Phase 5

- `P5.1` STT, VAD, And Turn Detection Adapters
- `P5.2` TTS Adapters And Voice Benchmark Harness

### Phase 6

- `P6.1` Tool Schema Layer And CRM Gateway
- `P6.2` Domain Tool Implementations

### Phase 7

- `P7.1` Sentiment And Escalation Policy Engine
- `P7.2` Session State, Audit Trail, And Analytics Events

### Phase 8

- `P8.1` Telephony Transport Adapter
- `P8.2` Pipecat Flow Assembly And FastAPI Server

### Phase 9

- `P9.1` Unit And Integration Harness
- `P9.2` Red-Team, Dialect, And Latency Evaluation Suite

### Phase 10

- `P10.1` Docker, Local Compose, And Cloud Templates
- `P10.2` Runbooks, Alerts, And Compliance Operations Pack
