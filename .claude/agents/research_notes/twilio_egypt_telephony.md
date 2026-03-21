# Research Note: Twilio & Egypt Telephony

## Summary

Twilio has significant gaps for Egypt: no local geographic phone numbers, no Middle East edge location, relatively high pricing ($0.17-0.19/min). Telnyx is the stronger alternative with local Egyptian numbers, own infrastructure, and lower pricing. Egyptian compliance requires explicit call recording consent, NTRA licensing, and PDPC approval for cross-border data transfers.

## Key Findings

### Twilio Egypt Gaps

- No local Egyptian geographic DIDs — only toll-free (+20800)
- No Middle East edge location — nearest is Frankfurt
- Egypt call pricing: $0.17-0.19/min (expensive)
- WebSocket/Media Streams: no separate charge (included in per-minute voice pricing)

### Twilio Media Streams Protocol

- Bidirectional WebSocket via `<Connect><Stream url="wss://..." />`
- Audio: mulaw 8kHz, base64-encoded in JSON messages
- Events: connected → start → media (continuous) → stop
- Send back: media (base64 mulaw), mark (playback position), clear (flush buffer for interruption)
- One bidirectional stream per call, one WebSocket per stream

### Pipecat Integration (Works Well)

- FastAPIWebsocketTransport + TwilioFrameSerializer
- Auto-converts between Twilio mulaw and Pipecat PCM
- Handles DTMF events as InputDTMFFrame
- Both dial-in and dial-out supported

### Better Alternative: Telnyx

- Local Egyptian phone numbers available ($1 setup + $1/month)
- Owns private global IP backbone
- Licensed telecom operator in 30+ countries
- 25-45% cheaper than Twilio on voice
- Also has Pipecat frame serializer
- **Recommendation:** Use Telnyx for Egypt telephony

### Other Alternatives

- **Etisalat (e&) CPaaS:** Carrier-grade, lowest local latency (local Egyptian infrastructure)
- **Vonage:** 85+ countries, WebSocket support
- **Plivo:** 70+ countries, $0.01/min starting

### Latency Concerns for Egypt

- No Twilio edge in Middle East — nearest is Frankfurt (~30-60ms RTT)
- Must colocate AI services in EU-West/Frankfurt or Middle East cloud (AWS Bahrain, GCP Doha)
- Twilio's own guide targets 1,115ms mouth-to-ear total
- ConversationRelay claims <500ms median, ~1000ms in practice

### Call Control Features

- Transfer: Conference bridge pattern — cold (remove original) or warm (3-party)
- Hold/resume: Conference Participants Resource with Hold=true/false
- Recording: record-from-start, programmatic start/stop/pause
- DTMF: <Gather> TwiML or WebSocket events

### Egyptian Compliance (CRITICAL)

- **NTRA License Required:** Call center operations need NTRA registration
- **Recording Consent:** Must obtain explicit consent (Articles 57-58 Constitution + Law 151/2020)
- **Data Protection Law 151/2020:** Voice recordings = personal data, explicit consent under Article 6
- **Cross-Border Transfer:** Requires PDPC approval, adequate protection standard
- **Data Retention:** Telecom Law requires 180 days storage with confidentiality
- **Penalties:** EGP 500K-5M for unlicensed processing, EGP 300K-3M for security breaches

## Impact on Architecture

- Use Telnyx instead of Twilio for Egypt telephony
- Host AI services in Frankfurt or Middle East cloud region
- Implement consent mechanism before call recording
- Budget for NTRA licensing and PDPC approval
- Plan for 180-day data retention requirement
