# Research Note: Deepgram STT For Egyptian Arabic

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: first STT integration

## Verified From Official Docs

- Deepgram `nova-3` supports `ar-EG`
- the language is available through the model-and-language configuration path

## Working Assumption

- Deepgram `nova-3` with `ar-EG` is the default first STT implementation because it has clear official language support and fits the project’s adapter-first design

## What Still Needs Project Testing

- real Egyptian call-center accuracy on domain vocabulary
- code-switching behavior
- endpointing and interruption behavior under live call pacing
- latency in the project’s actual hosting region

## Architectural Implication

- the STT package should support interim and final transcript events
- keyword or custom-vocabulary hooks should be exposed in config even if the first version uses minimal tuning

## Sources

- Deepgram model and language support docs
