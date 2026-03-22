# Research Note: Pipecat Framework

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: flow orchestration and service integration

## Safe Assumptions

- Pipecat is the target orchestration framework for the first implementation.
- Pipecat supports the architecture style the project needs: streaming audio, service adapters, flow graphs, and WebSocket-based hosting.
- Current Pipecat docs include Anthropic support at the service level, so the older claim that Anthropic is simply unsupported should not be carried forward as a blanket rule.

## Working Guidance

- keep Pipecat-specific code inside the pipeline package
- hide provider details behind app-level interfaces where possible
- use flows to model handoffs and node-local tool scopes

## Risks

- Pipecat is still evolving quickly, so version pinning matters
- provider-specific examples do not remove the need for project-owned interfaces

## Architectural Implication

- `P8.2` should assemble flows from already-stable contracts
- avoid spreading Pipecat primitives across every package

## Sources

- Pipecat docs and examples
- current source-of-truth architecture in `PROMPT.md`
