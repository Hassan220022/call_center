# Egyptian AI Voice Agent — Research & Decision Hub

Production-grade, real-time AI voice agent for Egyptian call centers. Speaks native Egyptian Arabic (Ammiya), maintains <800ms end-to-end latency, uses a multi-sub-agent architecture with specialized domain agents orchestrated by a supervisor.

---

## Quick Navigation

- [Research Findings Summary](#research-findings-summary)
- [Validated Tech Stack](#validated-tech-stack)
- [Critical Changes from Original Plan](#critical-changes-from-original-plan)
- [Architecture Overview](#architecture-overview)
- [Research Notes (Read in Order)](#research-notes-reading-order)
- [5 Key Decisions to Make](#5-key-decisions-to-make)
- [Cost Estimates](#cost-estimates)
- [Egyptian Compliance Requirements](#egyptian-compliance-requirements)
- [Project Files](#project-files)

---

## Research Findings Summary

7 parallel research agents investigated every layer of the system. Here are the validated results:

| Layer                | Original Plan             | Research Finding                                                                | Validated Choice                                                                      |
| -------------------- | ------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **LLM (Sub-Agents)** | Claude Sonnet 4           | Claude is the **weakest** LLM for Arabic — Anthropic's own docs say "basic MSA" | **GPT-4o** (best Egyptian with prompting) or **Gemini 2.5 Pro** (best native dialect) |
| **LLM (Supervisor)** | GPT-4o-mini               | 400-650ms TTFT per turn — too slow, doubles latency                             | **Semantic embedding router** (~10ms) with LLM fallback only for ambiguous intents    |
| **STT**              | Deepgram Nova-3           | Confirmed: ~40% lower WER than competitors for Egyptian Arabic                  | **Deepgram Nova-3 ar-EG** (test Soniox as challenger)                                 |
| **TTS (Primary)**    | Cartesia Sonic-3          | No Egyptian dialect differentiation — generic "ar" only                         | **ElevenLabs Eleven v3** + Professional Egyptian Voice Clone                          |
| **TTS (Fallback)**   | ElevenLabs Flash v2.5     | Cartesia has 40ms TTFA — unbeatable for latency                                 | **Cartesia Sonic-3 Turbo** (fallback for latency-critical paths)                      |
| **Telephony**        | Twilio                    | No local Egyptian numbers, no Middle East edge, $0.17-0.19/min                  | **Telnyx** (local Egyptian numbers, own backbone, 25-45% cheaper)                     |
| **VAD**              | Silero VAD v5             | Confirmed: <1ms on CPU, configurable for Egyptian speech patterns               | **Silero VAD v5** + **SmartTurn** (ML-based turn detection)                           |
| **Framework**        | Pipecat                   | Confirmed: v0.0.106, production-ready, AR_EG language enum                      | **Pipecat** with **Pipecat Flows** for multi-agent                                    |
| **Multi-Agent**      | LLM-based supervisor      | Prompt-level tool restrictions fail ~70% of the time                            | **Pipecat Flows** (code-level tool isolation per node)                                |
| **Security**         | Regex injection detection | Arabic bypasses most safety filters ($37,500 bounty earned)                     | **Hybrid regex + ML classifier** (95.8% F1 vs 37% regex-only)                         |

---

## Critical Changes from Original Plan

### 1. Claude is OUT as primary LLM

Anthropic's own documentation describes Claude's Arabic as "basic Arabic interactions, focusing on Modern Standard Arabic." Every benchmark confirms it's the weakest of the big 3 for Arabic. The AL-QASIDA benchmark (Dec 2024) found Claude "acknowledged it could not handle dialect requests well."

**Replacement:** GPT-4o (best Egyptian dialect among commercial APIs with prompt engineering) or Gemini 2.5 Pro (best native Arabic dialect support, 16+ dialects).

### 2. Supervisor is NOT an LLM call (on most turns)

Two sequential LLM calls (supervisor + sub-agent) = 1000-2150ms. Unacceptable for <800ms target.

**Replacement:** Semantic embedding router (~10ms, 92-96% accuracy). Only invoke LLM for ambiguous intents. Only classify on first turn and when topic change detected — not every turn.

### 3. Telnyx replaces Twilio for Egypt

Twilio has no local Egyptian geographic phone numbers (only toll-free +20800), no Middle East edge location (nearest: Frankfurt), and high pricing.

**Replacement:** Telnyx — local Egyptian numbers ($1/month), own infrastructure backbone, 25-45% cheaper, has Pipecat frame serializer.

### 4. ElevenLabs v3 becomes primary TTS

Cartesia Sonic-3 has no Egyptian dialect differentiation — generic "ar". ElevenLabs Eleven v3 explicitly markets Cairene dialect support and Professional Voice Clone preserves dialect characteristics.

**Replacement:** ElevenLabs v3 (primary, best dialect quality) + Cartesia Turbo (fallback, 40ms TTFA).

### 5. Tool isolation must be code-level

The AGENTIF Benchmark found LLMs follow fewer than 30% of complex constraint instructions. Prompt-level tool restrictions are fundamentally unreliable.

**Replacement:** Pipecat Flows with per-node tool definitions (code-level enforcement).

### 6. Arabic-specific security layer is mandatory

Arabic is a viable bypass vector for most safety filters (one researcher earned $37,500 in bounties using Arabic/Thai language switching). Homoglyph attacks have 58.7% success rate.

**Addition:** Arabic text normalization pipeline + hybrid regex/ML injection detection + Arabic-aware output filtering.

### 7. Egyptian compliance is non-trivial

- NTRA license required for call center operations
- Explicit consent required before call recording (Constitution + Law 151/2020)
- Cross-border data transfer needs PDPC approval
- 180-day data retention requirement
- Penalties: EGP 500K-5M for violations

---

## Architecture Overview

```
Caller speaks (Egyptian Arabic)
     |
     v
[Telnyx SIP / WebSocket]
     |
     v
[Silero VAD v5 + SmartTurn]  ── Egyptian-tuned: threshold=0.4, min_silence=300ms
     |
     v
[Deepgram Nova-3 ar-EG]  ── Streaming, 150ms interim, keyterm boosting for Arabic
     |
     v
[Arabic Text Normalization]  ── Strip diacritics, normalize hamza, homoglyph detection
     |
     v
[Pre-LLM Security Layer]
  ├── Injection Detector (regex + ML classifier, 95.8% F1)
  ├── Manipulation Detector (keyword tracking outside LLM context)
  └── Sentiment Tracker (EgyBERT/MARBERT-based)
     |
     v
[Semantic Embedding Router]  ── ~10ms, 92-96% accuracy
  ├── "orders"     → Orders Node
  ├── "booking"    → Booking Node
  ├── "complaints" → Complaints Node
  ├── "billing"    → Billing Node
  ├── "general"    → Fallback Node
  └── "ambiguous"  → GPT-4o-mini classification fallback
     |
     v
[Pipecat Flows Node]  ── Per-node: system prompt + tool whitelist + context strategy
     |
     v
[GPT-4o Sub-Agent]  ── Egyptian Ammiya prompt engineering:
  │                     few-shot examples + Fusha→Ammiya table
  │                     + periodic dialect reinforcement
  │
  ├── Tool Calls → [Tool Guardrails] → [CRM Client / Twilio Actions]
  │                 Pydantic validation, rate limiting, least privilege
  │
  └── Response Text
        |
        v
[Dialect Quality Validator]  ── Nile-Chat-12B or EgyBERT
  │                              If >30% MSA markers → regenerate
  │
  v
[Post-LLM Output Filter]  ── Block leaked prompts, PII, API keys
     |
     v
[ElevenLabs Eleven v3 TTS]  ── Professional Egyptian Voice Clone
  │                              Fallback: Cartesia Sonic-3 Turbo (40ms TTFA)
  │
  v
[Telnyx Audio Out] → Caller hears Egyptian Arabic response
```

---

## Research Notes (Reading Order)

Read these in order. Each builds on the previous and informs the next decision.

### 1. LLM Selection (Start Here)

**File:** [`.claude/agents/research_notes/best_llm_for_egyptian_arabic.md`](.claude/agents/research_notes/best_llm_for_egyptian_arabic.md)

The most impactful decision. Covers:

- Tier list of every LLM for Egyptian Arabic (Nile-Chat, Gemini, GPT-4o, Qwen3, ALLaM, Jais, Claude, AceGPT)
- Arabic function calling comparison (Gemini leads at 71.5% ComplexFuncBench Audio)
- AISA-AR-FunctionCall framework (reduces Arabic parse failures from 87% to <1%)
- Production Arabic voice agents (Kalimna AI, VoiceInfra Egypt, Maqsam)
- 3 recommended production stacks (GPT-4o, Gemini, Self-Hosted)

**Key takeaway:** GPT-4o for best Pipecat integration + good Egyptian; Gemini 2.5 Pro for best native dialect + function calling. Claude is out.

---

### 2. Egyptian Arabic Prompt Engineering

**File:** [`.claude/agents/research_notes/egyptian_arabic_in_ai.md`](.claude/agents/research_notes/egyptian_arabic_in_ai.md)

How to make the LLM consistently speak Egyptian Ammiya. Covers:

- All major LLMs drift toward Fusha — structural bias from MSA-dominant training data
- Few-shot prompting is the single most effective technique (AL-QASIDA research)
- Complete Fusha→Ammiya vocabulary substitution table (15+ word pairs)
- Egyptian honorifics for call center context (hadretak, ya fandem, ya basha)
- Phone call opening/closing patterns in Egyptian Arabic
- Code-switching handling (Arabic/English mix)
- Sentiment analysis for Egyptian (sarcasm detection — Egyptians use humor when frustrated)
- Text normalization pipeline (hamza, taa marbuta, diacritics, numerals)
- Dialect detection tools (EgyBERT: 84.25% F1, CAMeL Tools, MARBERT)

**Key takeaway:** Bilingual system prompt (English instructions + Arabic few-shot examples + Fusha→Ammiya table) is the highest-impact technique.

---

### 3. Multi-Agent Architecture

**File:** [`.claude/agents/research_notes/multi_agent_voice_architecture.md`](.claude/agents/research_notes/multi_agent_voice_architecture.md)

How to route callers to the right sub-agent without killing latency. Covers:

- Semantic embedding router (~10ms) vs LLM supervisor (400-850ms) — router wins
- Supervisor should NOT run on every turn — only first turn + topic changes
- Context passing: RESET_WITH_SUMMARY + structured state dictionary
- Tool isolation must be code-level (prompt-level fails ~70%)
- Pipecat Flows: node-based state machine, per-node prompts/tools/context strategies
- Latency analysis: embedding router brings E2E to ~800-1050ms on subsequent turns
- Vapi Squads (240K calls/day) validates this pattern at scale
- LLMSwitcher: Anthropic NOT supported (only OpenAI, Google)

**Key takeaway:** Hybrid State Machine (Pipecat Flows) + Embedding Router. Not every turn needs classification.

---

### 4. Pipecat Framework

**File:** [`.claude/agents/research_notes/pipecat_framework.md`](.claude/agents/research_notes/pipecat_framework.md)

The pipeline framework that connects everything. Covers:

- v0.0.106 (Mar 2026), production-ready, BSD-2 licensed, maintained by Daily.co
- Pipeline architecture: Frame → FrameProcessor → Pipeline → PipelineTask → PipelineRunner
- Native AR_EG language enum in the framework
- Twilio/Telnyx integration via FastAPIWebsocketTransport + FrameSerializer
- VAD: Silero (<1ms CPU) + SmartTurn (ML turn detection, default since v0.0.102)
- Interruption handling: MinWordsUserTurnStartStrategy(min_words=2) for Arabic
- Tool calling: FunctionSchema + register_function pattern
- Latency: 500-800ms E2E typical
- Limitations: pre-1.0 API, horizontal scaling only, Anthropic NOT in LLMSwitcher

**Key takeaway:** Pipecat Flows is the right abstraction for multi-agent. Must use OpenAI or Google LLM services for switching.

---

### 5. Speech-to-Text (Deepgram)

**File:** [`.claude/agents/research_notes/deepgram_arabic_stt.md`](.claude/agents/research_notes/deepgram_arabic_stt.md)

STT provider validation. Covers:

- Nova-3 supports ar-EG (Egyptian Arabic) — 17 Arabic language variants
- ~40% lower WER than competitors across Egyptian, Gulf, Levantine, North African
- Streaming: interim results within 150ms, finals <300ms
- Keyterm boosting works for Arabic (طلب, حجز, ميعاد, فاتورة, شكوى)
- Endpointing tunable to 300ms for fast Egyptian speech
- Native Pipecat integration: DeepgramFluxSTTService with Language.AR_EG
- Pricing: $0.0077/min pay-as-you-go
- Soniox: claims highest overall Arabic accuracy — worth head-to-head test

**Key takeaway:** Deepgram Nova-3 ar-EG confirmed as primary STT. Test Soniox before committing.

---

### 6. Text-to-Speech (Arabic TTS)

**File:** [`.claude/agents/research_notes/arabic_tts_options.md`](.claude/agents/research_notes/arabic_tts_options.md)

Making the agent SOUND Egyptian, not like an Al Jazeera newsreader. Covers:

- Cartesia Sonic-3: 40ms TTFA but generic "ar" — no Egyptian dialect differentiation
- ElevenLabs Eleven v3: most expressive, explicit Egyptian Cairene dialect marketing
- Professional Voice Clone: 2-3 hours of Egyptian speaker audio for best results
- Azure Neural TTS: ar-EG-SalmaNeural/ShakirNeural, 78% pronunciation error reduction
- Open-source winners: Habibi (F5-TTS, outperformed ElevenLabs v3), NileTTS (XTTS v2)
- Prosody control: emotion tags (Cartesia best), vowel stretching, fillers
- The MSA problem: training data overwhelmingly formal Arabic

**Key takeaway:** ElevenLabs v3 + Professional Egyptian Clone (primary), Cartesia Turbo (fallback for latency).

---

### 7. Telephony & Egypt Compliance

**File:** [`.claude/agents/research_notes/twilio_egypt_telephony.md`](.claude/agents/research_notes/twilio_egypt_telephony.md)

How to get calls in/out of Egypt and stay legal. Covers:

- Twilio gaps: no local Egyptian DIDs, no Middle East edge, $0.17-0.19/min
- Telnyx: local Egyptian numbers ($1/month), own backbone, 25-45% cheaper
- Etisalat (e&) CPaaS: lowest local latency (Egyptian infrastructure)
- Media Streams protocol: bidirectional WebSocket, mulaw 8kHz, base64 JSON
- Call control: Conference bridge for transfers, hold/resume, recording
- Latency: must colocate AI services in Frankfurt or Middle East cloud
- NTRA license required for call center operations
- Recording consent: explicit consent required (Constitution + Law 151/2020)
- Data Protection Law 151/2020: 180-day retention, PDPC approval for cross-border
- Penalties: EGP 500K-5M for unlicensed processing

**Key takeaway:** Use Telnyx for telephony. Budget for NTRA licensing. Implement consent mechanism before recording.

---

### 8. Security Architecture

**File:** [`.claude/agents/research_notes/voice_agent_security.md`](.claude/agents/research_notes/voice_agent_security.md)

How to defend an Arabic voice agent. Covers:

- Arabic bypasses most safety filters ($37,500 bug bounty earned via language switching)
- Homoglyph attacks: 58.7% success rate (highest of all Unicode attacks)
- Regex alone catches only 23% — hybrid regex+ML achieves 95.8% F1
- 4-layer defense: pre-LLM scan, prompt hardening, post-LLM filter, tool validation
- Arabic text normalization pipeline (7 steps before security scanning)
- Social engineering defense: keyword tracking OUTSIDE LLM context
- Session security state machine: NORMAL → CAUTIOUS → ELEVATED → HIGH_RISK → TRANSFER
- Meta's "Rule of Two": agent has untrusted input [A] + sensitive data [B] → state changes [C] require human
- CRM indirect injection: sanitize all external data before LLM context
- TEAPOT methodology: first voice AI penetration testing framework

**Key takeaway:** 4-layer defense is mandatory. Arabic-specific normalization before all scanning. Financial actions always go to humans.

---

## 5 Key Decisions to Make

After reading the research notes, these are the decisions that shape the entire system:

### Decision 1: Primary Sub-Agent LLM

| Option             | Egyptian Quality               | Function Calling                        | Pipecat Integration          | Risk                                             |
| ------------------ | ------------------------------ | --------------------------------------- | ---------------------------- | ------------------------------------------------ |
| **GPT-4o**         | Good (with heavy prompting)    | Strong (BFCL top tier)                  | **Best** (most mature)       | May drift to MSA in long conversations           |
| **Gemini 2.5 Pro** | **Best** (16+ dialects native) | **Best** (71.5% ComplexFuncBench Audio) | Good (LLMSwitcher supported) | Reports of "dialect drift" and "choppy segments" |

### Decision 2: Intent Classification

| Option                        | Latency   | Accuracy | Cost           |
| ----------------------------- | --------- | -------- | -------------- |
| **Semantic embedding router** | ~10ms     | 92-96%   | Near-zero      |
| GPT-4o-mini LLM call          | 400-650ms | 90-95%   | ~$0.001/call   |
| Fine-tuned BERT classifier    | 10-20ms   | 95-98%   | Self-host cost |

### Decision 3: Telephony Provider

| Option         | Egyptian Numbers    | Pricing        | Latency                  |
| -------------- | ------------------- | -------------- | ------------------------ |
| **Telnyx**     | Yes (local DIDs)    | 25-45% cheaper | Own backbone             |
| Twilio         | No (toll-free only) | $0.17-0.19/min | Frankfurt edge           |
| Etisalat CPaaS | Yes (carrier-grade) | Contact sales  | **Lowest** (local infra) |

### Decision 4: Primary TTS

| Option            | Egyptian Dialect         | Latency (TTFA) | Voice Clone                  |
| ----------------- | ------------------------ | -------------- | ---------------------------- |
| **ElevenLabs v3** | Best (Cairene marketing) | ~75-150ms      | Professional (2-3 hrs audio) |
| Cartesia Turbo    | Generic "ar"             | **40ms**       | Instant (10-30s audio)       |
| Azure Neural      | ar-EG voices             | ~100-200ms     | Professional fine-tuning     |

### Decision 5: STT Provider

| Option              | Egyptian WER                   | Streaming Latency | Code-Switching         |
| ------------------- | ------------------------------ | ----------------- | ---------------------- |
| **Deepgram Nova-3** | ~40% lower than competitors    | 150ms interim     | Good                   |
| Soniox              | Claims highest Arabic accuracy | Real-time         | **Built-in detection** |

---

## Cost Estimates (Per-Minute)

| Component             | Provider                        | Cost/Min            |
| --------------------- | ------------------------------- | ------------------- |
| STT                   | Deepgram Nova-3                 | $0.0077             |
| LLM (Sub-Agent)       | GPT-4o (~500 tokens/turn)       | ~$0.005-0.01        |
| LLM (Router fallback) | GPT-4o-mini (rare)              | ~$0.001             |
| TTS                   | ElevenLabs v3 (~100 chars/turn) | ~$0.005-0.01        |
| Telephony             | Telnyx Egypt                    | ~$0.10-0.15         |
| **Total estimated**   |                                 | **~$0.13-0.18/min** |

Compare to: Human agent cost $0.25-0.42/min (based on $15-25/hr agent cost)

---

## Egyptian Compliance Requirements

| Requirement           | Law/Regulation                         | Action Needed                        |
| --------------------- | -------------------------------------- | ------------------------------------ |
| Call center license   | NTRA Framework                         | Apply for NTRA registration          |
| Recording consent     | Constitution Art. 57-58 + Law 151/2020 | Pre-call consent mechanism           |
| Data protection       | Law 151/2020 (Nov 2025 exec regs)      | DPO appointment, privacy policy      |
| Cross-border transfer | Law 151/2020 Art. 14                   | PDPC approval application            |
| Data retention        | Telecom Law 10/2003                    | 180-day storage with confidentiality |
| Breach notification   | Law 151/2020                           | 72-hour notification process         |
| Penalties             | Various                                | EGP 500K-5M per violation            |

---

## Project Files

```
call_center/
├── README.md                          # This file — research hub & decision guide
├── CLAUDE.md                          # Claude Code project guidance
├── AGENTS.md                          # Sub-agent compliance requirements
│
├── .claude/
│   ├── agents/
│   │   └── research_notes/            # All research from the 7-agent swarm
│   │       ├── best_llm_for_egyptian_arabic.md   # [READ 1st] LLM tier list
│   │       ├── egyptian_arabic_in_ai.md          # [READ 2nd] Prompt engineering
│   │       ├── multi_agent_voice_architecture.md # [READ 3rd] Architecture patterns
│   │       ├── pipecat_framework.md              # [READ 4th] Pipeline framework
│   │       ├── deepgram_arabic_stt.md            # [READ 5th] STT validation
│   │       ├── arabic_tts_options.md             # [READ 6th] TTS comparison
│   │       ├── twilio_egypt_telephony.md         # [READ 7th] Telephony + compliance
│   │       └── voice_agent_security.md           # [READ 8th] Security architecture
│   ├── context/                       # Task implementation docs (generated during build)
│   ├── skills/                        # Development patterns
│   └── worktrees/                     # Git worktree tracking
│
└── egyptian-voice-agent/              # Application code (to be generated)
    ├── src/
    │   ├── server.py                  # FastAPI + WebSocket handler
    │   ├── pipeline.py                # Pipecat pipeline assembly
    │   ├── config/                    # Settings, constants
    │   ├── prompts/                   # Egyptian Ammiya prompts
    │   ├── agents/                    # Supervisor + sub-agents
    │   ├── tools/                     # Tool definitions + handlers
    │   ├── security/                  # 4-layer defense
    │   ├── voice/                     # STT, TTS, VAD config
    │   ├── middleware/                # Manipulation detection, sentiment
    │   └── monitoring/               # Metrics, logging, alerts
    ├── tests/
    │   └── red_team/                  # Arabic injection + manipulation tests
    └── scripts/                       # Deployment, benchmarking
```

---

## Next Steps

1. **Read** the 8 research notes in order (above)
2. **Decide** the 5 key choices (LLM, router, telephony, TTS, STT)
3. **Approve** the validated design
4. **Write** the implementation spec (design doc)
5. **Generate** the implementation plan (phase-by-phase tasks)
6. **Build** — phase 1 through 10
