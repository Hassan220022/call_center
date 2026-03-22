# Research Note: Egyptian Arabic Prompting

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: prompt style and dialect regression tests

## Safe Assumptions

- General-purpose models tend to drift toward Fusha unless prompted and tested against it.
- A short, phone-call-oriented prompt style is more important than impressive long-form Arabic prose.
- Few-shot examples and explicit negative guidance against Fusha are both useful.

## Working Prompt Rules

- keep customer-facing text in Egyptian Ammiya
- keep replies to 1 to 3 sentences
- use respectful Egyptian honorifics when appropriate
- mirror light Arabic-English code switching when the caller does
- escalate instead of improvising around policy boundaries

## Signals To Test For

- Fusha markers such as formal connective phrases and stiff question forms
- overlong responses that sound like chat, not a phone call
- loss of Egyptian phrasing after several turns
- refusal language that becomes robotic or too formal

## Architectural Implication

- prompt QA fixtures are mandatory, not optional
- dialect quality belongs in tests, not just prompt review

## Implementation Link

- `P2.1` defines the base persona and prompt contract
- `P2.2` adds domain prompts and prompt QA fixtures
