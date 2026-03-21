# Research Note: Deepgram Arabic STT & Alternatives

## Summary

Deepgram Nova-3 supports Egyptian Arabic (ar-EG) as a first-class language with ~40% lower WER than competitors across Gulf, Egyptian, Levantine, and North African dialects. Streaming latency <300ms, interim results within 150ms. Keyterm boosting works for Arabic including mid-stream updates. Soniox is a strong challenger with highest overall Arabic accuracy in benchmarks. Pricing: $0.0077/min pay-as-you-go.

## Key Findings

### Nova-3 Arabic Support

- **Arabic is supported on Nova-3** — first RTL language on the platform
- **17 Arabic language variants** including ar-EG (Egyptian Arabic)
- Supports: Gulf, MSA, Egyptian, Levantine, North African, Iraqi, Maghrebi
- Language code: `ar` for general Arabic, `ar-EG` for Egyptian specifically
- **~40% lower WER** than competing STT systems on conversational Arabic
- "Consistently low WER across regions, outperforming competitors most clearly in dialect-heavy regions"

### Streaming Configuration

- **Interim results:** Delivered within 150ms, marked with `is_final: false`
- **Final results latency:** <300ms for streaming workloads
- **Endpointing:** VAD-based, configurable via integer parameter (milliseconds)
  - Can be tuned for fast Egyptian speech (e.g., 300ms)
  - Custom silence duration to detect end of speech
- **Measure latency using interim transcripts** — finals are delayed by endpoint detection

### Keyterm Prompting (Works for Arabic)

- `keyterm` parameter boosts recognition for specialized terms
- Can update mid-stream via `STTUpdateSettingsFrame` (no WebSocket reconnect needed)
- Start without keywords, add gradually — Deepgram may already perform well
- Can boost Arabic domain terms: طلب, حجز, ميعاد, فاتورة, شكوى
- For 200+ keywords, upgrade to trained AI model

### Pipecat Integration

- **Native `DeepgramSTTService`** and newer `DeepgramFluxSTTService`
- Install: `pip install pipecat-ai[deepgram]`
- Configuration via `DeepgramFluxSTTService.InputParams`:
  - `language` (e.g., Language.AR_EG)
  - `model` (e.g., "nova-3-general")
  - `keyterm` (list of boost terms)
  - `endpointing` (silence duration ms)
- On-the-fly settings update without reconnect: keyterm, language
- `ttfs_p99_latency` tunable for benchmarked latency values

### Pricing (2026)

| Plan          | Nova-3 Rate | Notes                             |
| ------------- | ----------- | --------------------------------- |
| Pay-As-You-Go | $0.0077/min | $200 free credits, no CC required |
| Growth        | $0.0065/min | $4,000/year prepaid, 20% savings  |
| Enterprise    | Custom      | Custom models, priority features  |

- Billed per second (not rounded to minute)
- Flux/Nova-3 Monolingual: $0.0065/min

## Alternatives Comparison

### Soniox (Strong Challenger)

- **Highest overall Arabic accuracy** in 2025 benchmarks
- Supports Egyptian Arabic + code-switching detection
- Real-time streaming for voice agents
- "Native-speaker accuracy" claimed
- Outperforms Deepgram, Google, OpenAI, Azure, AWS, AssemblyAI on Arabic
- **Worth testing** alongside Deepgram

### Google Cloud Speech-to-Text

- Supports Arabic including ar-EG locale
- Multiple Arabic dialect models available
- Good accuracy but Deepgram claims 40% lower WER
- Higher pricing than Deepgram

### Azure Speech Services

- 140+ languages including Arabic dialects
- ar-EG (Egyptian Arabic) supported
- Real-time and batch processing
- Enterprise compliance (SOC 2, HIPAA)

### AssemblyAI

- Arabic support via Universal-2 model
- Good general accuracy but not dialect-specific
- Less Arabic dialect differentiation than Deepgram

### Whisper (Self-Hosted)

- Supports Arabic (trained on 680K hours multilingual data)
- Good for code-switching but inconsistent for dialects
- Fine-tuning on Egyptian data (Whisper-Medium-Egyptian) improves accuracy
- Higher latency than Deepgram streaming (~1-3 seconds)

### Qwen3-ASR (New Entrant)

- Alibaba's ASR model supporting multilingual speech recognition
- Includes Arabic with language detection
- Self-hosted option
- Worth monitoring but less proven for Egyptian dialect

## Recommendation

**Primary STT: Deepgram Nova-3 with ar-EG**

- Best documented Egyptian dialect support with WER benchmarks
- Native Pipecat integration (DeepgramFluxSTTService)
- Sub-300ms streaming latency
- Keyterm boosting for Arabic domain vocabulary
- Production-proven at scale

**Test Against: Soniox**

- Claims highest Arabic accuracy overall
- Code-switching detection built-in
- Run a head-to-head comparison with Egyptian Arabic audio samples before committing

## Sources

- [Nova-3 Arabic STT](https://deepgram.com/learn/nova-3-arabic-speech-to-text-production-grade-stt)
- [Deepgram Language Support](https://developers.deepgram.com/docs/language)
- [Deepgram Endpointing Docs](https://developers.deepgram.com/docs/understand-endpointing-interim-results)
- [Deepgram Pipecat Integration](https://docs.pipecat.ai/server/services/stt/deepgram)
- [Deepgram Pricing](https://deepgram.com/pricing)
- [Soniox Arabic Voice Agents](https://soniox.com/use-cases/voice-agents/arabic)
- [Soniox Benchmarks](https://soniox.com/benchmarks)
- [Deepgram Keyterm Expansion](https://deepgram.com/learn/deepgram-expands-nova-3-with-10-new-languages-and-multilingual-keyterm-prompting)
