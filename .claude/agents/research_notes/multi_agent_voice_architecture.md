# Research Note: Multi-Agent Voice Architecture

## Summary

Research validates the Supervisor → Sub-Agent pattern but recommends replacing the LLM-based supervisor with an embedding-based semantic router (~10ms) to eliminate 400-850ms of latency. Pipecat Flows provides the best implementation primitive. Tool isolation must be enforced at code level — prompt-level restrictions fail ~70% of the time.

## Key Findings

### Supervisor Pattern — Use Semantic Router, NOT LLM

- **Embedding router** (e.g., aurelio-labs/semantic-router): ~10ms, 92-96% accuracy
- **Fine-tuned BERT classifier**: 10-20ms, 95-98% accuracy
- **GPT-4o-mini**: 400-650ms TTFT — too slow for every turn
- **Claude Haiku 4.5**: 637ms median TTFT — too slow for every turn
- **Recommendation:** Cascading approach: keyword filter → embedding router → LLM fallback only for ambiguous

### Supervisor Frequency — NOT Every Turn

- First turn: Run supervisor to classify initial intent
- Subsequent turns: Route directly to active sub-agent
- Re-invoke supervisor ONLY when user signals topic change (keyword/embedding distance check)
- This eliminates supervisor latency from 90%+ of turns

### Context Passing Between Agents

- Best strategy: RESET_WITH_SUMMARY on agent switches + structured state dictionary
- From Vapi Squads: Transfer only user/assistant messages, filter out system/tool messages
- Sparse follow-ups ("yes", "ok") should stay with current agent

### Latency Analysis (Two-LLM-Call Problem)

- Sequential supervisor + sub-agent: 1000-2150ms — unacceptable
- With embedding router + streaming: ~900-1260ms first turn, ~800-1050ms subsequent
- Mitigations: prefix caching (-200-400ms), filler phrases, streaming overlap

### Tool Isolation — Code-Level ONLY

- AGENTIF Benchmark: LLMs follow fewer than 30% of complex constraint instructions
- "Control Illusion" paper: prompt-level instruction hierarchy fundamentally unreliable
- Answer.AI: LLMs hallucinate tool calls to tools not provided to them
- **Must enforce tool whitelists in code** via Pipecat Flows per-node tool definitions

### Pipecat Flows Implementation

- Each node = agent with own role_messages, task_messages, functions, context_strategy
- Transitions via function handlers returning next node config
- Global state dictionary persists across transitions
- LLMSwitcher for runtime provider switching (OpenAI/Google only, not Anthropic)

### Claude vs GPT for Voice

- Claude Sonnet 4.6: 100% accuracy on voice benchmark, 850ms TTFT, top-3 function calling
- BUT: Arabic support is "basic MSA" — weakest major LLM for Arabic
- GPT-4o: Better Arabic but voice output "largely incomprehensible" for dialects
- Gemini: Best Arabic dialect support (16+ dialects) but less mature voice pipeline

### Recommended Architecture

Hybrid State Machine + Embedding Router:

1. Model core flows as Pipecat Flows graph (deterministic transitions)
2. Embedding router for initial intent classification (~10ms)
3. LLM fallback (Haiku) only for ambiguous intents
4. Each node has own system prompt + tool set (code-level isolation)

## Sources

- Vapi Squads (240K calls/day), Retell AI, OpenAI Agents SDK
- Microsoft Azure AI Agent Orchestration Patterns
- OWASP AI Agent Security Cheat Sheet
- Berkeley Function Calling Leaderboard V4
