# Research Note: LLM Selection For Egyptian Arabic

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: text-model benchmark plan, not final provider lock

## Safe Assumptions

- Egyptian-Ammiya quality is not reliably proven by vendor marketing alone.
- Official multilingual claims do not equal strong Egyptian-dialect performance in this exact call-center use case.
- The project should keep LLM access behind an adapter and benchmark at least two providers before production lock.

## Current Working Position

- The implementation spec uses an OpenAI-compatible adapter as the first path because it lowers integration risk.
- A Gemini adapter remains the main challenger for dialect quality benchmarking.
- Anthropic should not be treated as “ruled out by docs.” Anthropic’s current multilingual docs present Arabic as a strong supported language, so any rejection must come from project benchmarks, not outdated summaries.

## What Must Be Benchmarked

- Egyptian naturalness over multi-turn call-center exchanges
- Fusha drift under longer dialogs
- tool-call correctness and recovery behavior
- short-answer compliance
- escalation and refusal style in Egyptian Ammiya

## Architectural Implication

- Keep provider access behind a common adapter.
- Do not hard-code prompt or routing logic to a single model vendor.
- Treat provider choice as benchmark-gated until `P9.2`.

## Primary Sources

- Anthropic multilingual support docs
- project-specific benchmark plan in `PROMPT.md`
