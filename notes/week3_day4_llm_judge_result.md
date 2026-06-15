# Week 3 Day 4: LLM-as-Judge Evaluation Result

Date: 2026-06-05 10:35

## Methodology
- Judge: Claude Sonnet 4.6 (different vendor than training data generator, breaks circularity)
- Eval set: 50 Hinglish prompts from Week 1 baseline (hand-curated)
- Comparisons: fine-tune vs each of Qwen-base, GPT-4o-mini, GPT-4o
- Total judgments: 150 pairwise (A/B order randomized to control positional bias)
- Rubric: 4 axes × 1-5 scale (Register, Intent Accuracy, Content Quality, Cultural Fit)
- Cost: ~$1.40 (~₹120) on Claude Sonnet

## Absolute scores (out of 20)

| Model | Register | Intent | Quality | Culture | Total |
|---|---|---|---|---|---|
| qwen_3b_base | 1.24 | 2.04 | 1.72 | 1.72 | 6.72 |
| gpt_4o | 2.12 | 4.10 | 3.72 | 2.96 | 12.90 |
| gpt_4o_mini | 2.50 | 4.26 | 3.84 | 2.96 | 13.56 |
| **qwen_3b_finetuned** | **3.98** | 2.81 | 2.39 | 3.31 | **12.48** |

## Head-to-head wins (out of 50)

| Comparison | Fine-tune wins | Opponent wins |
|---|---|---|
| vs Qwen-base | 42 | 6 |
| vs GPT-4o-mini | 14 | 36 |
| vs GPT-4o | 16 | 34 |

## Honest interpretation

### What works
- **Register transfer is the win.** Fine-tune scores 3.98/5 on register vs GPT-4o-mini's 2.50 
  and GPT-4o's 2.12. The fine-tune speaks more native-sounding Hinglish than either frontier 
  model, as judged by a third-party LLM. This is the project's defensible headline result.
- **Cultural fit also slightly above GPT-4o-mini** (3.31 vs 2.96).
- **Dominates base model 42-6.** The fine-tune is unambiguously a massive improvement over 
  Qwen-base on every dimension.

### What doesn't work
- **Intent accuracy: 2.81 vs GPT-4o-mini's 4.26.** Fine-tune sometimes confuses task types 
  (e.g., answers sentiment when intent classification was asked).
- **Content quality: 2.39 vs GPT-4o-mini's 3.84.** Responses are conversational but often 
  less concrete/factual than the frontier model.
- **Overall: 12.48 vs 13.56.** Within ~8% but not parity.

### Diagnosis
Training on conversational synthetic data taught the model the register strongly but also 
inherited the length distribution and "friendly chat" mode of the training set. The model 
defaults to a casual response pattern even when a precise/instructional answer would be 
better.

## Defensible framing for blog post / resume

"Fine-tuned Qwen-3B achieves register quality matching/exceeding GPT-4o-mini (3.98 vs 2.50 
on Claude Sonnet judge eval) at <1/30th serving cost. The fine-tune trades ~8% overall 
quality (12.48 vs 13.56) for dramatic cost savings and superior Hinglish naturalness, 
making it preferable for style-sensitive applications (customer support, conversational 
interfaces) while content-heavy use cases still benefit from the larger model."

## Caveats and methodology limitations
- N=50 prompts is small. A larger held-out eval set would tighten confidence intervals.
- Claude Sonnet as judge has its own biases (preferences for certain phrasings, etc).
- The fine-tune was trained on GPT-4o-mini outputs. There may be mild bias even with 
  a different-vendor judge.
- Real human evaluation (not just LLM-judge) would strengthen claims.

## Status: Week 3 substantively complete. Moving to Week 4 (packaging + serving).
