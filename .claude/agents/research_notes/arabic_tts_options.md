# Research Note: Arabic TTS Options for Egyptian Voice Agent

## Summary

The biggest TTS challenge is making Arabic sound like Egyptian Ammiya, not MSA newsreader. ElevenLabs Eleven v3 + Professional Voice Clone is the best commercial option. Cartesia Sonic-3 Turbo has lowest latency (40ms TTFA) but may lean MSA. Open-source Habibi (F5-TTS) outperformed ElevenLabs v3 in dialect benchmarks.

## Key Findings

### Cartesia Sonic-3

- Arabic supported (42 languages), but NO Egyptian dialect differentiation — just generic "ar"
- **Sonic Turbo: 40ms TTFA** — unbeatable for real-time
- Voice cloning: 3 seconds minimum, 10-30s recommended
- 60+ emotion tags, SSML speed/volume control mid-utterance
- Native Pipecat integration (CartesiaTTSService, WebSocket + HTTP)
- Risk: Arabic output may lean MSA; language=None for auto-detection
- Pricing: ~$0.011/1K chars, Startup plan $49/month

### ElevenLabs

- Flash v2.5: ~75ms TTFA, 32 languages with Arabic (Saudi/UAE locales)
- **Eleven v3: Most expressive, 70+ languages, explicit Egyptian dialect marketing**
- Professional Voice Clone: 2-3 hours of audio for best results
- Controls: stability (0-1), similarity_boost (0-1), style (0-1), speed (0.7-1.2x)
- Native Pipecat integration (ElevenLabsTTSService)
- Pricing: Pro plan $99/month for 500K credits, Flash models 0.5 credits/char

### Azure Neural TTS (Strong Enterprise Option)

- Egyptian Arabic voices: ar-EG-SalmaNeural (F), ar-EG-ShakirNeural (M)
- 78% reduction in pronunciation errors (recent improvement)
- Professional voice fine-tuning for ar-EG
- Personal Voice creation supported
- Best compliance/SLA story

### Open Source Winners

- **Habibi (F5-TTS based):** Outperformed ElevenLabs v3 on dialect benchmarks, 20+ Arabic dialects
- **NileTTS (XTTS v2 fine-tuned):** 49.4% CER reduction for Egyptian Arabic, trained on 38.1 hours
- **SawtArabi:** First benchmark corpus for dialectal Arabic TTS (Interspeech 2025)

### NOT Recommended

- Amazon Polly: No Egyptian Arabic (ar-EG), only MSA and Gulf
- Google Cloud TTS: Has ar-EG locale but "truly localized dialect options can be patchy"

### The MSA Problem

- Training data overwhelmingly MSA (news, audiobooks)
- Arabic NLP tools (tokenizers, phonemizers) built for MSA
- Egyptian Ammiya has no standardized orthography
- Solutions: Write input in dialect spelling, clone Egyptian voice, fine-tune open-source models

### Prosody Control for Egyptian

- Vowel stretching (يا سلاااام!): Some models interpret repeated chars as elongation — test per provider
- Fillers (يعني..., اممم...): Include explicitly in LLM output text
- Emotion: Cartesia best (60+ tags), ElevenLabs v3 "most expressive"
- Speed changes: Cartesia supports mid-utterance SSML, ElevenLabs per-request only

## Recommended Stack

- **Primary TTS:** ElevenLabs Eleven v3 + Professional Egyptian Voice Clone (best dialect quality)
- **Fallback TTS:** Cartesia Sonic-3 Turbo (40ms TTFA for latency-critical paths)
- **Future:** Self-host Habibi/F5-TTS for best dialect quality + full control
