# Research Note: Arabic TTS Options

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: TTS benchmark plan, not final provider lock

## Safe Assumptions

- Egyptian-sounding TTS quality cannot be inferred safely from generic Arabic support pages alone.
- The project needs both a quality benchmark and a latency benchmark.
- TTS must stay behind a provider-neutral adapter.

## Working Position

- benchmark ElevenLabs for cloned-voice quality
- benchmark Cartesia for low-latency behavior
- keep Azure `ar-EG` as an enterprise-oriented fallback path

## What Must Be Measured

- Egyptian naturalness
- Fusha drift
- interruption recovery
- time to first audio
- stability under repeated short responses

## Architectural Implication

- `P5.2` owns both adapters and the benchmark harness
- no prompt or business logic may depend on vendor-specific TTS features

## Sources

- official provider docs for setup and capabilities
- project benchmark gates in `PROMPT.md`
