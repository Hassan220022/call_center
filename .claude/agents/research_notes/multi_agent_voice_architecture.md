# Research Note: Multi-Agent Voice Architecture

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: agent runtime, routing, and handoff design

## Safe Assumptions

- Domain isolation should be enforced in code through per-agent tool allowlists.
- Routing should avoid an extra full LLM call on every turn when a deterministic or embedding-based path is good enough.
- Handoffs should transfer summarized state, not the full raw conversation every time.

## Working Architecture Shape

- router picks the active domain
- each domain agent owns its own prompt and tools
- handoffs carry compact state and summaries
- risky session state lives outside model context

## What Still Needs Project Testing

- how accurate the first-turn router is on real Egyptian call-center data
- when topic change should trigger re-routing
- how much context can be removed before user experience degrades

## Architectural Implication

- prioritize `P3.1` and `P3.2` before pipeline assembly
- avoid a monolithic all-tools agent

## Sources

- Pipecat flows documentation
- internal architecture decisions in `PROMPT.md`
