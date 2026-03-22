# Research Note: Voice Agent Security

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: security package and red-team test design

## Safe Assumptions

- model prompts alone are not enough to protect tools or business policies
- Arabic normalization must happen before security scanning
- manipulation and escalation state should live outside model context
- output should be filtered before TTS

## Working Security Shape

1. normalize text
2. scan input for injection risk
3. track manipulation across turns
4. validate tool access and tool input/output
5. filter model output before speech generation

## What Needs Testing

- Arabic-script prompt injection
- homoglyph and control-character abuse
- Arabizi variants
- indirect injection through CRM or tool data
- escalation behavior after repeated policy abuse

## Architectural Implication

- `P4.1` and `P4.2` are core tasks, not polish work
- `P9.2` must include security regression cases from the start

## Sources

- project security rules in `PROMPT.md`
- future benchmark and red-team outputs
