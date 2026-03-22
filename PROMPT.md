# PROMPT.md

# Egyptian AI Voice Agent

Canonical architecture and delivery specification.

Last reviewed: March 22, 2026

## 1. Mission

Build a real-time voice agent for Egyptian call centers that:

- speaks natural Egyptian Ammiya
- keeps spoken answers short
- uses least-privilege tool access per domain agent
- escalates risky situations instead of improvising
- is instrumented, testable, and production-ready

This document is the source of truth for implementation unless it is explicitly superseded.

## 2. Current Starting Point

The repository starts from a planning state.

What exists:

- architecture docs
- task plan
- research notes

What does not exist yet:

- Python project scaffold
- provider adapters
- FastAPI service
- Pipecat flows
- tests and deployment files

The first implementation work must create that baseline.

## 3. Product Constraints

### 3.1 Language

- All customer-facing prompts and responses must be in Egyptian Ammiya.
- Avoid Fusha-heavy wording.
- The assistant may mirror common Arabic-English code switching when the caller does.
- Answers must stay short enough for voice calls, typically 1 to 3 sentences.

### 3.2 Safety

- Financial state changes require a human.
- Unverifiable claims about what another agent promised require a human.
- Manipulation state is tracked outside the LLM context.
- Tool access is enforced in code, not by prompt wording alone.
- Output passes through a filter before TTS.

### 3.3 Operational

- The system must support streaming audio and interruptions.
- The system must emit structured logs, metrics, and audit events.
- Production telephony and data-handling choices require Egypt-specific legal review.

## 4. Decision Register

| Area | Decision | Status | Notes |
| --- | --- | --- | --- |
| Python runtime | Python 3.12+ with `uv` | Locked | Standardize on one toolchain |
| Code layout | Root project with `src/egyptian_voice_agent/` | Locked | No nested app repo |
| Web layer | FastAPI session server | Locked | WebSocket-based voice session handling |
| Voice orchestration | Pipecat flows plus adapters | Locked | Multi-agent flow graph |
| Customer language | Egyptian Ammiya only | Locked | No Fusha in customer-facing strings |
| Domain design | Orders, booking, complaints, billing, fallback | Locked | Separate prompts and tool scopes |
| Tool enforcement | Code-level allowlists per agent | Locked | No prompt-only enforcement |
| STT | Deepgram `nova-3` with `ar-EG` first | Locked | First provider to implement |
| Routing pattern | Deterministic plus embedding router, with LLM fallback | Locked | Keep critical path fast |
| Security shape | Normalization, injection detection, manipulation tracker, output filter | Locked | All outside-prompt protections remain mandatory |
| Telephony integration style | Provider adapter layer | Locked | Avoid hard-coding a single vendor |
| Dev telephony | Twilio Media Streams | Provisional | Strong docs and easy local start |
| Egypt production telephony | Telnyx or another Egypt-compliant provider | Needs legal review | Must pass commercial and regulatory validation |
| Primary text LLM | OpenAI-compatible adapter first | Provisional | Initial implementation path |
| Challenger text LLM | Gemini adapter benchmark | Needs benchmark | Evaluate dialect quality and tool behavior |
| Primary TTS | Benchmark-driven between ElevenLabs PVC and Cartesia | Needs benchmark | Quality vs latency tradeoff |
| Enterprise fallback TTS | Azure `ar-EG` option remains available | Provisional | Compliance-oriented backup |

## 5. Target Architecture

```text
Caller Audio
  -> Telephony Transport Adapter
  -> VAD / Turn Detector
  -> STT Adapter
  -> Arabic Normalization
  -> Pre-LLM Security Checks
  -> Intent Router
  -> Active Domain Agent
  -> Tool Guardrails
  -> Post-LLM Output Filter
  -> TTS Adapter
  -> Caller Audio Out
```

### 5.1 Core Components

#### Telephony Transport Adapter

- abstracts vendor-specific WebSocket message formats
- normalizes audio input/output events
- handles call lifecycle, interruption, marks, and transfer hooks

#### VAD And Turn Detection

- captures user turn start and end
- supports interruption of assistant audio
- exposes timing controls for experimentation

#### STT Adapter

- streams interim and final transcripts
- defaults to `ar-EG`
- supports runtime boosting and tuning hooks

#### Arabic Normalization

- strips unsafe control characters
- normalizes Arabic variants for matching and security scanning
- preserves enough raw context for auditability

#### Pre-LLM Security Checks

- prompt injection detector
- manipulation tracker
- risk-state machine

These checks happen before model execution and do not rely on the model to protect itself.

#### Intent Router

- starts with deterministic and embedding-based routing
- falls back to an LLM only for ambiguous cases
- avoids a second LLM hop on every turn when possible

#### Domain Agents

Each domain agent has:

- its own system prompt
- its own tool allowlist
- its own response constraints
- access only to the minimum state it needs

Domains:

- orders
- booking
- complaints
- billing
- fallback

#### Tool Guardrails

- validate schemas
- enforce authorization and policy
- rate limit and timeout tool calls
- sanitize external data before it reaches model context

#### Post-LLM Output Filter

- blocks prompt leakage
- blocks secrets and unsafe output
- enforces short spoken responses
- can trigger regeneration or escalation

#### TTS Adapter

- supports primary and fallback providers
- keeps a voice profile contract independent of provider
- measures time-to-first-audio and total synthesis latency

## 6. Prompt Architecture

### 6.1 Global Prompt Rules

All domain prompts must enforce:

- Egyptian Ammiya only
- short phone-call responses
- clear refusal boundaries
- human transfer when policy requires it

### 6.2 Prompt Layers

The prompt system should be split into:

1. prompt contract and variable schema
2. shared persona and tone rules
3. domain-specific instructions
4. short prompt QA fixtures

### 6.3 Prompt Style Rules

- use natural Egyptian phrasing
- use respectful honorifics
- avoid long explanations
- avoid promising unavailable actions
- prefer clear next steps over empathy-only filler

## 7. Agent Runtime Design

### 7.1 Agent Core

Create a shared interface that all agents implement.

Minimum responsibilities:

- receive normalized conversation state
- expose a tool allowlist
- generate a response under short-answer constraints
- emit structured routing and escalation metadata

### 7.2 Router Design

Use a hybrid strategy:

- deterministic first-pass rules for obvious domains
- embedding similarity for fuzzy cases
- LLM fallback only for ambiguity or topic change

Do not require a separate LLM classification call on every turn.

### 7.3 Context Strategy

Use short context windows and summarized handoff state:

- active-domain memory
- last meaningful user turns
- tool outcome summaries
- risk state

Avoid replaying full tool and system history across handoffs.

## 8. Security Architecture

### 8.1 Required Layers

1. Arabic normalization
2. input security scanning
3. manipulation tracking
4. tool validation
5. output filtering

### 8.2 Risk States

Use an explicit state machine such as:

- `NORMAL`
- `CAUTIOUS`
- `ELEVATED`
- `HIGH_RISK`
- `TRANSFER`

The risk state must be stored outside model context and survive across turns.

### 8.3 Mandatory Human Escalation Cases

- financial changes or waivers
- unverifiable prior-agent promises
- repeated policy-manipulation attempts
- confirmed injection abuse
- requests that exceed tool or policy scope

## 9. Voice Stack Design

### 9.1 STT

Initial provider:

- Deepgram `nova-3`
- language `ar-EG`

The STT interface must support:

- interim transcripts
- final transcripts
- keyword boosting hooks
- latency measurements

### 9.2 TTS

Implement provider-neutral TTS contracts first.

Benchmark candidates:

- ElevenLabs professional voice clone path
- Cartesia low-latency path
- Azure `ar-EG` fallback path

Do not hard-code business logic to any one TTS SDK.

### 9.3 Voice Benchmarks

Every TTS decision must be evaluated on:

- Egyptian naturalness
- MSA drift
- interruption behavior
- latency
- stability under streaming usage

## 10. Telephony Strategy

### 10.1 Adapter Requirement

Telephony must sit behind an adapter so the rest of the system does not depend on vendor-specific event payloads.

### 10.2 Development Path

Start with Twilio Media Streams for the first full dev loop because:

- the docs and examples are mature
- Pipecat integration is straightforward
- it lowers bootstrap risk

### 10.3 Production Path

Before production, validate:

- Egypt-specific numbering and routing
- NTRA and cloud call-center requirements
- recording-consent flow
- cross-border handling constraints

The production telephony provider is not considered locked until that work is done.

## 11. Observability

Instrument from the start.

Required outputs:

- structured JSON logs
- correlation IDs per session and call
- latency metrics by stage
- tool call metrics
- escalation and transfer reasons
- audit events for consent and policy boundaries

## 12. Testing Strategy

The system is not done without tests for:

- unit behavior of core contracts
- prompt regression and Fusha drift
- Arabic normalization and injection handling
- tool-allowlist enforcement
- domain routing
- end-to-end streaming call smoke tests
- latency budgets

### 12.1 Dialect Tests

Dialect quality is a first-class product requirement.

Tests must check:

- Fusha markers
- sentence count and length
- Egyptian honorific use
- code-switch behavior
- policy-safe refusal style

### 12.2 Security Tests

Security suites must include:

- Arabic prompt injection
- homoglyph payloads
- Arabizi variants
- indirect injection through tool or CRM data
- manipulation and escalation flows

## 13. Target Repository Layout

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

## 14. Swarm Execution Model

The swarm should work contract-first and package-first.

### 14.1 Execution Rules

- freeze interfaces before provider-heavy work
- keep tasks narrow and ownership boundaries clear
- prefer adapters over direct SDK coupling
- integrate in thin slices, not one big merge at the end

### 14.2 First Waves

Wave 1:

- `P1.1` architecture freeze and ADR capture

Wave 2:

- `P1.2` repository bootstrap foundation

Wave 3:

- `P2.1` prompt contract and base persona
- `P4.1` Arabic normalization and injection detection
- `P5.1` STT, VAD, and turn detection adapters

Wave 4 onward:

- prompts, agent runtime, security, voice, and tool contracts split in parallel
- middleware and session-state work follow once runtime and policy layers exist
- telephony transport and flow assembly happen only after adapters and policy boundaries are stable

## 15. Definition Of Done

The project is not done when it merely answers a demo call.

It is done when:

- the codebase exists and runs locally
- prompts stay in Egyptian Ammiya
- tool isolation is enforced in code
- policy violations escalate correctly
- at least one end-to-end telephony path works
- tests cover dialect, security, and core call flows
- metrics and logs are present
- deployment and ops artifacts exist
- legal and compliance items are explicitly tracked

## 16. What Must Not Happen

Do not:

- hard-code prompt-only tool restrictions
- lock production to a telephony vendor before legal review
- claim benchmark wins without project-specific testing
- let research notes overrule the implementation spec
- ship long chatbot-style speech
- allow the AI to process refunds, credits, discounts, or unverifiable exceptions
