# Research Note: Telephony And Egypt Constraints

## Status

- Supporting note
- Last reviewed: March 22, 2026
- Decision impact: dev transport vs production telephony decision

## Verified From Primary Sources

- Egypt has a formal NTRA call-center framework
- cloud call-center usage requires prior written consent from the authority in that framework
- telephony and recording behavior cannot be treated as a pure engineering choice

## Working Position

- use a provider adapter in all cases
- start development with the easiest documented streaming path
- do not lock production to a provider until numbering, routing, cost, and legal review are complete

## What Still Needs Verification

- exact Egypt-local number availability for candidate providers
- production routing quality from the target hosting region
- recording-consent and cross-border handling requirements in the final operating model

## Architectural Implication

- `P8.1` must normalize provider events behind one interface
- production-provider choice remains a gated decision outside early implementation tasks

## Sources

- NTRA call-center framework
- telephony vendor documentation for streaming transports
