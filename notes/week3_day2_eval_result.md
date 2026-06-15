# Week 3 Day 2: Training complete + automated eval

Date: 2026-06-03 15:27

## Training (qlora-r16-2ep)
- Final loss: 0.95
- Wall clock: 9.2 minutes on Blackwell RTX PRO 6000 (95 GB)
- Throughput: 38.6 samples/sec
- Peak GPU memory: 10.24 GB
- LoRA adapter size: 125.2 MB
- W&B run: 3yenkm1g

## Automated diagnostic on Week 1 baseline 50-prompt set

| Model | Hing% | EnglDrift% | DevanagDrift% |
|---|---|---|---|
| qwen_3b_base | 8.9% | 32.0% | 12.5% |
| gpt_4o_mini | 29.5% | 0.0% | 0.0% |
| gpt_4o | 24.6% | 4.0% | 2.5% |
| **qwen_3b_finetuned** | **31.6%** | **0.0%** | **0.0%** |

## Headline finding
Fine-tuned Qwen-3B matches GPT-4o-mini and exceeds GPT-4o on Hinglish marker 
density, while eliminating both the English-drift and Devanagari-injection 
failure modes of the base model.

The Hinglish density gap (8.9 → 31.6) closed in full — the fine-tune is now 
slightly more Hinglish-dense than even GPT-4o-mini, which is intuitive: a 
model trained specifically on Hinglish instruction data should outperform a 
generalist model on register-matching, even at much smaller scale.

## Caveat
Automated metrics measure *style* (does it speak Hinglish?), not *quality* 
(is the response useful, accurate, contextually appropriate?). The hand-scored 
qualitative evaluation comes next.

## Next
- Qualitative spot-check (5 random prompts × 4 models, side-by-side)
- LLM-as-judge eval using Claude Sonnet as the non-circular judge
- Write Week 3 results notes
