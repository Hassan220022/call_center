# Research Note: Best LLM for Egyptian Arabic Voice Agent

## Summary

No single model is perfect for Egyptian Arabic + function calling + voice. The recommended production stack is a **tiered approach**: GPT-4o for sub-agent responses (best Egyptian dialect among commercial APIs with prompt engineering), semantic router for intent classification (no LLM needed), and Nile-Chat-12B as a dialect quality validator. Gemini 2.5 Pro is the strongest alternative if willing to use Google's ecosystem.

## Tier List: Egyptian Arabic LLM Quality

### Tier 1: Best Egyptian Dialect Generation

**1. Nile-Chat-12B (MBZUAI)**

- Purpose-built for Egyptian Arabic (Ammiya)
- 3 variants: 4B dense, 3x4B-A6B MoE, 12B dense
- Trained on 3.3B tokens of Egyptian web text + 1.9M instruction samples
- Supports both Arabic script AND Arabizi (Latin-script Egyptian)
- Outperforms LLaMA, ALLaM, Jais on Egyptian benchmarks
- Open-source on HuggingFace (MBZUAI-Paris/Nile-Chat-12B)
- **Limitation:** Small model (12B), no native function calling, not an API service
- **Best use:** Dialect quality validator, not primary generation LLM

**2. Gemini 2.5 Pro (Google)**

- Supports 16+ Arabic dialects including Egyptian
- Best native dialect handling among commercial APIs
- "More natural, culturally aware" than GPT-4o in Arabic testing
- Gemini 2.5 Flash TTS explicitly supports "Arabic (Egypt)"
- Function calling: 71.5% on ComplexFuncBench Audio (leads the field)
- 90% instruction adherence rate
- **Limitation:** Reports of "dialect drift" and "choppy segments" in preview
- **Best use:** Primary LLM if using Google ecosystem

**3. GPT-4o (OpenAI)**

- Good Egyptian Arabic text generation with heavy prompt engineering
- Best for dialects with "larger training representation" (Egyptian and Levantine)
- Strong function calling (Berkeley BFCL benchmark)
- Mature Pipecat integration
- **Limitation:** Voice output for Arabic dialects "largely incomprehensible" (text is fine, voice is not)
- **Best use:** Primary sub-agent LLM for TEXT generation (not voice-native)

### Tier 2: Good But Not Egyptian-Specific

**4. Qwen3-235B-A22B (Alibaba)**

- Top open-source for Arabic broadly, 119 languages
- Strong tool/agent capabilities
- Includes Ta'izzi-Adeni Arabic dialect
- No specific Egyptian dialect optimization documented
- **Best use:** Self-hosted alternative if GPT-4o/Gemini not viable

**5. ALLaM-7B (SDAIA/IBM)**

- Saudi Arabic LLM, supports several dialects including Egyptian
- 72-74% accuracy on AraLingBench
- Good at understanding Egyptian but inconsistent at generating it
- **Best use:** Gulf Arabic applications, not Egyptian-primary

**6. Jais-2-70B (TII/Abu Dhabi)**

- Highest scoring on Emirati dialect benchmark (Alyah)
- Strong general Arabic but Gulf-focused
- **Best use:** Gulf Arabic applications

### Tier 3: Avoid for Egyptian Arabic

**7. Claude Sonnet 4 (Anthropic)**

- "Basic Arabic interactions, focusing on MSA" — official Anthropic docs
- Weakest major LLM for Arabic dialects
- **Not recommended** as primary LLM for Egyptian Arabic voice agent

**8. AceGPT**

- Lowest scores on dialect benchmarks (38.88% on Absher)
- **Not recommended**

## Arabic Function Calling

### Commercial APIs

- **Gemini 2.5 Pro:** 71.5% on ComplexFuncBench Audio — BEST for Arabic function calling in voice
- **GPT-4o:** Strong function calling, mature ecosystem, well-tested with Pipecat
- **Claude Sonnet 4.6:** Top-3 on Berkeley BFCL (70.29%) but Arabic quality is weak

### Specialized Framework

- **AISA-AR-FunctionCall:** 270M-parameter framework specifically for Arabic function calling
  - Reduces parse failures from 87% to <1%
  - Improves function name accuracy by 8x
  - Enhances argument alignment across dialects
  - Open-source under AISA framework
  - Could be used as a function call parser layer

## Production Voice Agents Using Arabic

### Kalimna AI

- First Arabic-native AI voice platform (launched 2025)
- 95% accuracy across 6 Gulf dialects
- 100+ businesses served
- $0.15/min pricing
- **GCC-focused (not Egyptian)**

### VoiceInfra Egypt

- AI voice agents deployed in Egyptian retail/telecom
- Uses specialized models, not just GPT-4o

### Maqsam

- Saudi-founded with Cairo subsidiary
- Their Arabic ASR was "a disaster" initially — shows the difficulty

## Recommended Production Stack

### Option A: Best Quality (Recommended)

```
Intent Classification: Semantic embedding router (~10ms)
  ↓ (fallback for ambiguous)
  GPT-4o-mini or Gemini Flash (fast classification)

Sub-Agent LLM: GPT-4o
  - Heavy Egyptian Ammiya prompt engineering
  - Few-shot examples in system prompt
  - Fusha-to-Ammiya vocabulary table
  - Periodic dialect reinforcement

Dialect Validator: Nile-Chat-12B or EgyBERT
  - Run on every LLM response
  - If MSA detected (>30% MSA markers), regenerate with stronger prompting

Function Calling: GPT-4o native
  - Mature, reliable, well-tested with Pipecat
```

### Option B: Google Ecosystem

```
Intent Classification: Semantic router + Gemini Flash fallback
Sub-Agent LLM: Gemini 2.5 Pro (best native dialect support)
Dialect Validator: Nile-Chat-12B
Function Calling: Gemini native (71.5% on ComplexFuncBench Audio)
```

### Option C: Maximum Control (Self-Hosted)

```
Intent Classification: Fine-tuned BERT classifier (~10-20ms)
Sub-Agent LLM: Qwen3-235B-A22B (self-hosted)
Dialect Validator: Nile-Chat-12B
Function Calling: AISA-AR-FunctionCall layer
```

## Key Insight

The critical decision is GPT-4o vs Gemini 2.5 Pro:

- **GPT-4o:** More mature Pipecat integration, proven function calling, good Egyptian with prompting
- **Gemini 2.5 Pro:** Better native Arabic dialect support, best audio function calling, but LLMSwitcher support in Pipecat, potential dialect drift issues
- **Both need:** Egyptian voice clone for TTS (neither model's voice output is good for Egyptian)

The LLM generates TEXT that goes to TTS — so text quality in Egyptian dialect matters more than the LLM's own voice capabilities.

## Sources

- [Nile-Chat MBZUAI](https://huggingface.co/MBZUAI-Paris/Nile-Chat-12B)
- [Nile-Chat Paper](https://arxiv.org/abs/2507.04569)
- [Gemini Arabic Dialect Quality](https://arabie.ai/en/blog/2025-10-24-gemini-is-quietly-crushing-chatgpt-and-claude-for-arabic-and-nobody-noticed)
- [Claude Arabic Limitations](https://arabie.ai/en/blog/2025-10-24-chatgpt-vs-claude-for-arabic-which-ai-is-better-in-2025)
- [AISA-AR-FunctionCall](https://arxiv.org/abs/2603.16901)
- [aiXplain Arabic LLM Benchmark](https://aixplain.com/arabic-llm-benchmark-report/)
- [DialectalArabicMMLU Benchmark](https://arxiv.org/abs/2510.27543)
- [Kalimna AI](https://kalimna.ai/)
- [GPT-4o Arabic Dialect Issues](https://community.openai.com/t/issue-with-gpt-4-voice-chat-handling-of-arabic-dialects/785941)
- [Qwen3 Arabic Support](https://qwenlm.github.io/blog/qwen3/)
- [Arabic LLM Benchmarks List](https://github.com/tiiuae/Arabic-LLM-Benchmarks)
- [Cross-dialectal Arabic Translation LLM Comparison](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2025.1661789/full)
- [Gemini 2.5 Audio Function Calling](https://blog.google/technology/google-deepmind/gemini-2-5-native-audio/)
