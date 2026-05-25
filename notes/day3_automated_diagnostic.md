# Day 3: Automated Diagnostic — Baseline Gap on 50 Hinglish Prompts

**Date:** Week 1, Day 3
**Notebook:** `notebooks/01_baseline_gap_analysis.ipynb`
**Outputs file:** `data/baseline_outputs.json`

## Goal

Before hand-scoring, run automated proxies on the 50-prompt baseline outputs from Qwen2.5-3B-Instruct, GPT-4o-mini, and GPT-4o. The goal is a quick, reproducible signal of where the gap is — which informs the hand-scoring rubric and the eventual fine-tuning data mix.

## Eval set composition

50 hand-curated Hinglish prompts across 4 intent categories:

| Category | Count |
|---|---|
| casual | 10 |
| customer support | 10 |
| question | 20 |
| sentiment-analysis | 10 |

Script distribution: 40 Roman-script, 10 mixed-script (Roman + Devanagari).

All prompts authored by hand — no GPT-generated examples, to keep this as gold ground truth.

## Three automated proxies

| Metric | What it measures | Computed how |
|---|---|---|
| **Hinglish marker density** | Whether the model responds in Hinglish vs drifts to English | % of words in the response that are common Hindi/Urdu function words (`hai`, `mein`, `kar`, `ke`, `aur`, `bhai`, `yaar`, etc., ~40 markers) |
| **English drift rate** | Whether the model opens in English meta-text instead of replying in Hinglish | % of responses whose first 150 chars contain English giveaways (`I understand`, `It seems`, `Here are`, `As an AI`, etc.) |
| **Devanagari injection rate** | Whether the model switches to Devanagari script when the input was pure Roman | % of Roman-input prompts whose response contains any Devanagari character (U+0900–U+097F) |

These are crude proxies — they don't measure intent understanding, factual accuracy, or cultural fit. Hand scoring (Day 4) addresses those. But they're a fast, reproducible signal of the most basic question: *does the model even speak Hinglish?*

## Results

| Model | Hinglish marker % | English drift % | Devanagari injection % |
|---|---|---|---|
| **Qwen2.5-3B-Instruct (base)** | **8.9%** | **32.0%** | **12.5%** |
| GPT-4o-mini | 29.2% | 0.0% | 0.0% |
| GPT-4o | 24.6% | 4.0% | 2.5% |

## Observations

**1. Qwen2.5-3B barely produces Hinglish.**
At 8.9% Hindi-marker density, Qwen-base is responding mostly in English. GPT-4o-mini sits at 29.2% — roughly the natural density you'd expect from native Hinglish. The gap is ~20 percentage points.

**2. Qwen drifts to English in ~1/3 of responses.**
32% of Qwen-base responses open with English meta-text — *"I understand that..."*, *"It seems like..."*, *"Yes, ..."* — instead of replying in the user's register. GPT-4o-mini never does this (0%); GPT-4o does it occasionally (4%).

**3. Qwen randomly injects Devanagari.**
On 5 of 40 Roman-script prompts (12.5%), Qwen-base produced responses containing Devanagari characters. This is a script-handling bug, not a stylistic choice. GPT-4o-mini never injects Devanagari for Roman inputs (0%); GPT-4o does so rarely (2.5%).

**4. The gap is concentrated in conversational and customer-support categories.**
36 of 50 prompts triggered very low Hinglish density (< 15%) from Qwen-base. These cluster in casual conversation, customer support, and questions — exactly the use cases where natural Hinglish matters most.

## Sample failures

**P002, casual — "Chalo let's go to eat lunch, shaam hone wali hai?"**
- Qwen: *"Shaaam (also spelled as Shaam or Shom) in Hindi and Urdu refers to the evening mealtime, which is typically around 6:00 PM to 8:00 PM..."* — drifts into a dictionary entry, English explanation
- GPT-4o-mini: *"Haan, chalo! Lunch karna zaroori hai, aur shaam hone wali hai toh achha hai ki hum thoda jaldi nikalte hain. Kahan chalna hai?"* — natural Hinglish reply

**P012, customer support — "Yaar Swiggy wala galat order le aaya, kya karu?"**
- Qwen: *"कोनाहिल यहाँ से मदद करूँ। आपको एक नया अनुचेतनामूल्य खरीदने के लिए कैसे कर सकता हूँ?..."* — flips to Devanagari, hallucinates a nonsensical Hindi word ("anuchetanaamoolya")
- GPT-4o-mini: *"Agar aapka Swiggy par order galat aaya hai, to aap in steps ko follow kar sakte hain..."* — direct, useful, Hinglish

**P022, question — "Bhaiya yeh apple kitne mai diya?"**
- Qwen: Hinglish density 0% — full English response, no Hinglish at all
- GPT-4o-mini: Stays in Hinglish, addresses the question

## Implications for the project

1. **The gap is real, large, and measurable.** Qwen2.5-3B-base needs help on basic register-matching before it can be a useful Hinglish assistant. The fine-tune has clear room to improve.

2. **Script handling is its own failure mode.** Beyond just "speaks Hinglish or doesn't," the model has a bug where it incorrectly switches to Devanagari. The fine-tuning data should explicitly cover both scripts, and the eval set should track script consistency.

3. **GPT-4o-mini is a credible target.** It's not perfect (24-29% marker density isn't 100%, and individual outputs occasionally miss intent), but it's clearly competent at Hinglish conversational generation. Matching its quality at 3B scale + 30× lower serving cost is the headline thesis.

4. **Categories matter for data mix.** Casual conversation and customer support show the largest Qwen failures. The instruction-tuning dataset (Week 2) should weight these categories heavily.

## Caveats

- 50 prompts is small. The automated proxies are noisy and the differences between GPT-4o-mini and GPT-4o on these metrics are within sample variance.
- Hindi marker density is a crude proxy — a model could score high on it while still producing bad responses. Day 4 hand scoring addresses this.
- All prompts were authored by one annotator (me). They reflect one person's intuition about what Hinglish looks like. Broadening to more annotators would strengthen the eval.

## Next

- Day 4: Hand-score all 150 outputs on 4 axes (Register / Intent / Accuracy / Culture, 1–5 each) using a blinded scoring sheet.
- Week 2: Use the failure patterns above to inform dataset construction — explicitly cover Roman + Devanagari scripts, weight casual and customer-support categories heavily.
- Week 4 evaluation: Re-run these same automated proxies on the fine-tuned model to measure gap closure. The 8.9% → ??? change on Hinglish marker density will be one of the headline numbers.
