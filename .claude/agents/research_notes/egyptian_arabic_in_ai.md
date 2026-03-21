# Research Note: Egyptian Arabic (Ammiya) in AI Systems

## Summary

Comprehensive research on how LLMs handle Egyptian colloquial Arabic. **Critical finding:** All major LLMs drift toward Modern Standard Arabic (Fusha), with Claude being the weakest for Egyptian dialect. GPT-4o and Gemini 2.5 Pro perform better with heavy prompt engineering. Few-shot prompting is the single most effective technique for maintaining dialect.

## Key Findings

### LLM Egyptian Arabic Quality (AL-QASIDA Benchmark, Dec 2024)

- **Core problem:** LLMs understand dialectal Arabic better than they produce it. Strong reluctance to generate dialect; default to MSA.
- **GPT-4o:** Recommended for cross-lingual Egyptian Arabic. Has documented pattern of insisting on MSA but can be managed with prompt engineering.
- **Claude Sonnet 4:** Acknowledged it cannot handle dialect requests well. Defaults to MSA with awkward phrasing. Described as "basic MSA" compared to competitors.
- **Gemini 2.5 Pro:** Supports 16+ Arabic dialects vs GPT's 2-3. Scored 88.6% on multilingual tasks. Better dialect handling and cultural context.
- **Nile-Chat (MBZUAI):** Most significant Egyptian-specific model (4B/12B). Designed for both Arabic script AND Arabizi. Outperforms LLaMA, Jais, ALLaM on Egyptian benchmarks. Available on HuggingFace.

### Fusha Drift

All LLMs drift toward Fusha in longer conversations due to MSA-dominant training data. Manifests as vocabulary formalization, formal grammar, and formal discourse markers.

### Best Prompt Engineering Techniques

1. **Few-shot prompting** — single most effective technique (from AL-QASIDA research)
2. **Bilingual system prompt** — English instructions + Arabic few-shot examples + Fusha-to-Ammiya vocabulary table
3. **Explicit dialect instruction** — `"قم بالرد بالعامية المصرية"` + `"لا تستخدم الفصحى أبدًا"`
4. **Vocabulary anchoring table** (Fusha → Ammiya substitutions):
   - سأقوم → هعمل
   - يمكنني → أقدر
   - هل → إيه (questions)
   - ماذا → إيه
   - لذلك/إذن → عشان كده/يبقى
   - أريد → عايز/عايزة
   - الآن → دلوقتي
   - كيف → إزاي
5. **Periodic dialect reinforcement** — inject reminders every 5-10 turns

### Dialect Detection Tools

- **EgyBERT** — Best Egyptian-specific model. 84.25% F1, 87.33% accuracy. Trained on 10.4GB Egyptian text.
- **CAMeL Tools** (NYU Abu Dhabi) — Open-source Python toolkit with dialect ID across Egyptian, Gulf, Levantine, NA, MSA.
- **MARBERT** — 160M params, pretrained on 128GB dialectal Arabic (1B tweets).

### Key Fusha vs Ammiya Markers

| Feature    | Fusha (MSA)   | Egyptian Ammiya |
| ---------- | ------------- | --------------- |
| Future     | سوف / سَـ     | هـ (prefix)     |
| "What"     | ماذا          | إيه             |
| "How"      | كيف           | إزاي            |
| "Now"      | الآن          | دلوقتي          |
| "This"     | هذا/هذه       | ده/دي           |
| "Want"     | أريد          | عايز/عايزة      |
| "Because"  | لأن/لذلك      | عشان/عشان كده   |
| "Very"     | جداً          | أوي/قوي         |
| "Not"      | لم/لن/لا      | مش / ما...ش     |
| "Can"      | يمكنني/أستطيع | أقدر            |
| "Go"       | أذهب          | أروح            |
| "There is" | يوجد/هناك     | فيه             |
| Relative   | الذي/التي     | اللي            |

### Egyptian Honorifics for Call Center

- **حضرتك (hadretak/hadretik)** — Default. Formal "you" like French "vous". Safest choice.
- **يا فندم (ya fandem)** — Formal, respectful. Neutral gender.
- **يا باشا (ya basha)** — Friendly-respectful. Primarily male.
- **يا هانم (ya hanem)** — Respectful for women, like "madam".
- **يا أستاذ/أستاذة (ya ostaz/ostaza)** — Professional "Mr./Ms."

### Phone Call Pattern

Opening: `"السلام عليكم، أهلاً بحضرتك في [company]، معاك [name]، إزاي أقدر أساعد حضرتك؟"`
Closing: `"فيه حاجة تانية أقدر أساعد حضرتك فيها؟"` then `"شكراً لحضرتك، يوم سعيد إن شاء الله"`

### Code-Switching

- Egyptians commonly mix Arabic/English, especially technical/business vocabulary
- Agent should mirror caller's code-switching level
- ArzEn corpus: 12 hours of Egyptian Arabic-English code-switched speech
- Whisper handles code-switching reasonably; fine-tuning on ArzEn improves it

### Sentiment Analysis

- **EgyBERT** or **MARBERT** fine-tuned for Egyptian sentiment — best approach
- **NileULex** — Egyptian Arabic sentiment lexicon
- **Critical:** Egyptians use humor/sarcasm when frustrated. `"يا سلام!"` can mean opposite.
- Combine text sentiment + acoustic features + contextual cues

### Text Normalization Pipeline

1. Unicode normalization (NFKC)
2. Remove tatweel/kashida (ـ)
3. Normalize Alef variants → ا
4. Normalize Alef Maksura (ى) → ي
5. Normalize Taa Marbuta (ة) → ه (for matching)
6. Remove diacritics/harakat
7. Normalize Eastern Arabic numerals to Western
8. Tools: PyArabic (fast normalization), CAMeL Tools (comprehensive + dialect ID)

### Existing Egyptian Arabic AI Projects

- **Maqsam** — Saudi-founded with Cairo subsidiary. Arabic ASR historically "a disaster" due to dialect diversity.
- **FutureBee AI** — Egyptian Arabic telecom call center dataset (40+ hrs real conversations)
- **VoiceInfra Egypt** — AI voice agents in Egyptian retail/telecom

## Impact on Tech Stack

**CRITICAL:** The plan specifies Claude Sonnet 4 as primary LLM for sub-agents. Research shows Claude is the WEAKEST option for Egyptian Arabic. Consider:

1. **GPT-4o** as primary (best Egyptian dialect with prompt engineering)
2. **Gemini 2.5 Pro** as alternative (best native dialect support, 16+ dialects)
3. **Claude** only for English-language tasks or as fallback
4. **Nile-Chat-12B** for Egyptian-specific classification/validation tasks

## Related Tasks

- Phase 2: All prompt files (base_prompt, sub_agent_prompts)
- Phase 4: Security layer (Arabic injection patterns)
- Phase 5: Voice services (STT/TTS Arabic configuration)
- Phase 9: Tests (dialect quality checks)
