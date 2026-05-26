# Day 4: Hand-Scored Baseline — The Gap is 12 Points on a 20-Point Scale

**Date:** Week 1, Day 4
**Inputs:** `data/baseline_outputs.json` (150 model outputs), `data/baseline_scored.csv` (150 hand-scored rows)
**Annotator:** 1 (project author, native Hinglish speaker)

## Method

For each of the 150 (prompt × model) outputs, scored on 4 axes (1–5 each):

| Axis | What it measures |
|---|---|
| Register | Does the response speak natural Hinglish at the right tone? |
| Intent | Did the model understand what the user wanted? |
| Accuracy | Are the facts and claims correct? |
| Culture | Does the response feel like an Indian context (idioms, references, examples)? |

**Total = sum of 4 axes, max 20.**

Plus a single `failure_mode` tag for the most prominent failure when total < 16. Vocabulary:
`hallucinated_words`, `language_drift_english`, `language_drift_devanagari`, `intent_miss`, `hallucinated_facts`, `ai_disclaimer`, `cultural_mismatch`, `tone_mismatch`, `meta_text_open`, `none`.

Scoring was conducted with the `model` column hidden (rows shuffled within each prompt group), so model identity did not bias individual scores.

## Headline result

| Model | Mean / 20 | Std | Min | Max |
|---|---|---|---|---|
| **Qwen2.5-3B-Instruct (base)** | **7.78** | 4.26 | 4 | 14 |
| GPT-4o-mini | 19.84 | 0.89 | 14 | 20 |
| GPT-4o | 19.50 | 2.02 | 10 | 20 |

**12.06-point gap** between Qwen2.5-3B-base and GPT-4o-mini on a 20-point scale. This is the gap the fine-tune aims to close.

## Per-axis breakdown

| Model | Register | Intent | Accuracy | Culture |
|---|---|---|---|---|
| Qwen-base | **1.08** | 2.46 | 2.44 | 1.80 |
| GPT-4o-mini | 4.98 | 4.90 | 5.00 | 4.96 |
| GPT-4o | 4.82 | 4.82 | 5.00 | 4.86 |

**Register is the dominant failure axis for Qwen-base** at 1.08/5 — essentially the lowest possible. The model isn't producing bad Hinglish; it's not producing Hinglish at all. Intent and Accuracy are mid-low, suggesting that when Qwen does try to answer, it frequently misses or hallucinates.

This is informative for the fine-tune: register/style transfer is exactly what SFT on instruction-formatted Hinglish data should fix. Intent and Accuracy should improve as a side-effect of the model staying in the right register.

## Per-category breakdown

| Category | Qwen | GPT-4o-mini | GPT-4o | Gap (mini − Qwen) |
|---|---|---|---|---|
| casual | 7.50 | 19.20 | 17.50 | 11.70 |
| customer support | 9.10 | 20.00 | 20.00 | 10.90 |
| question | 6.05 | 20.00 | 20.00 | **13.95** |
| sentiment-analysis | 10.20 | 20.00 | 20.00 | 9.80 |

The "question" category shows the largest gap (13.95 points). It is also where the mixed-script prompts live (10 of 20 question prompts use mixed Roman+Devanagari input) — Qwen handles these especially poorly. The training data mix in Week 2 should weight question-answering prompts in both scripts heavily.

## Failure mode distribution (Qwen-base)

| Failure mode | Count | % of failures |
|---|---|---|
| hallucinated_words | 23 | 46% |
| language_drift_english | 17 | 34% |
| intent_miss | 4 | 8% |
| hallucinated_facts | 4 | 8% |
| cultural_mismatch | 1 | 2% |
| language_drift_devanagari | 1 | 2% |

**80% of Qwen failures concentrate in two modes: word hallucination and English drift.** Both are addressable by SFT on high-quality Hinglish instruction data — neither requires exotic training techniques (DPO, RLHF). This is encouraging for the project's feasibility on the planned compute budget.

## Where the target itself is weak (the upside)

GPT-4o-mini scored < 20 on only 2 of 50 prompts:
- **P002** (casual, total=18): *"Chalo let's go to eat lunch, shaam hone wali hai?"* — partial intent miss
- **P004** (casual, total=14): *"Movie kaisi lagi, did you enjoy it?"* — triggered an AI-disclaimer ("Mujhe movies dekhne ka experience nahi hota...")

GPT-4o scored < 20 on 3 prompts, all in casual conversation, all triggered by AI-disclaimer behavior:
- **P004** (total=10)
- **P007** (total=12) — *"Weekend par kya kiya? IPL ka match dekha it was mindblowing"* → *"Main AI hoon, isliye main IPL nahi dekh sakta..."*
- **P008** (total=13) — *"Office ke baad I'm thinking to go to swim, tum bhi chaloge kya"* → AI disclaimer about being virtual

**Interesting finding: GPT-4o is *worse* than GPT-4o-mini on casual Hinglish conversation (17.50 vs 19.20).** This appears to be because GPT-4o is more aggressive about issuing AI-identity disclaimers, which break the casual register. A fine-tune that learns natural casual conversation without AI-disclaimers could plausibly *beat* GPT-4o on this subset.

## Concrete examples of the gap

**P001 — casual, "Bhai weekend pe Bangalore mein kya karein?"**
- Qwen (total=7): *"Bhai, Bangalore ke weekend mein ek aur ek options hai - mall se chalna ya city se jyada travel karne ke liye. Bangalore ka bazaar miliyonis ke saath hi baap aur maam se badal kar..."* — hallucinated words ("miliyonis"), garbled syntax
- GPT-4o-mini (total=20): *"Bangalore mein weekend par bahut saari activities karne ke liye hain. 1. **Nandi Hills**: Subah subah Nandi Hills ki taraf jaakar sunrise dekhna..."* — specific local recommendations, clean Hinglish

**P012 — customer support, "Yaar Swiggy wala galat order le aaya, kya karu?"**
- Qwen (total=4): *"कोनाहिल यहाँ से मदद करूँ। आपको एक नया अनुचेतनामूल्य खरीदने के लिए कैसे कर सकता हूँ?..."* — flips to Devanagari, hallucinates "anuchetanaamoolya" (not a real word)
- GPT-4o-mini (total=20): *"Agar aapka Swiggy par order galat aaya hai, to aap in steps ko follow kar sakte hain: 1. App kholen... 2. Order history..."* — concrete, actionable

**P004 — casual, "Movie kaisi lagi, did you enjoy it?"**
- GPT-4o (total=10): *"I'm an AI, so I don't have personal experiences..."* — AI disclaimer breaks the casual register
- GPT-4o-mini (total=14): better, but still mentions "Mujhe movies dekhne ka experience nahi hota"
- Qwen (total=4): hallucinated a fake movie called "Kaisi Hai Ya Kya Na", attributed it to wrong director and stars

## Implications for Week 2 (dataset construction)

1. **Primary objective: register transfer.** The fine-tuning data should be high-quality Hinglish instruction-following examples. Translation pairs alone won't do it — the model needs to learn to *generate* in Hinglish conversationally.

2. **Data mix weighting:**
   - Heavy weight on **question-answering** (biggest gap, 14 points)
   - Strong weight on **customer support** (second-biggest practical gap)
   - Moderate weight on **casual conversation** (smaller gap but where GPT-4o-mini itself is weakest — opportunity to exceed target)
   - Light weight on **sentiment/classification** (smallest gap, already partially addressed by GLUECoS-style data)

3. **Script coverage is critical.** Include both pure-Roman and mixed-Roman+Devanagari prompts. Qwen's 12.5% Devanagari-injection rate on Roman inputs is a script-handling bug that explicit dual-script training should fix.

4. **Avoid AI-disclaimers in training data.** Generating synthetic Hinglish examples via GPT-4 risks inheriting GPT-4's tendency to issue AI-identity disclaimers. Filter these out at the data construction stage, or prompt GPT-4 to roleplay as a human Indian assistant.

## Caveats

- 50 prompts, single annotator. Inter-annotator agreement is unmeasured. A second annotator on a 20-prompt subset would strengthen the eval.
- Hand scoring is subjective even with a rubric. Numbers should be read as ordinal signal, not precise quality measurement.
- The held-out test set for Week 4 evaluation will be larger (200 prompts) and use LLM-as-judge for scalability — this 50-prompt hand-scored set serves as the calibration anchor.

## Resume-worthy summary

> *On a 50-prompt hand-scored Hinglish evaluation set across casual, customer support, question-answering, and sentiment categories, Qwen2.5-3B-Instruct (base) scored 7.78/20 vs GPT-4o-mini's 19.84/20. The gap was driven primarily by language register (Qwen: 1.08/5, GPT-4o-mini: 4.98/5), with 80% of Qwen failures classified as hallucinated_words or language_drift_english. GPT-4o was surprisingly weaker than GPT-4o-mini on casual conversation (17.50 vs 19.20) due to more frequent AI-identity disclaimers breaking the Hinglish register.*
