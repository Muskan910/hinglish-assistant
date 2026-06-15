# Week 3 Day 1: Sanity training milestone

Date: 2026-06-03 15:08

## Setup
- GPU: NVIDIA RTX PRO 6000 Blackwell (95 GB VRAM)
- Base model: Qwen2.5-3B-Instruct (4-bit, NF4 quantization)
- LoRA: rank=16, alpha=32, target_modules=all linear layers
- Trainable params: ~30M (~1% of base)

## Sanity training
- 100 examples × ~5 epochs × 30 steps
- Batch size 8, gradient accumulation 2 (effective batch 16)
- Learning rate 2e-4, bf16, linear schedule
- Wall clock: ~5 minutes

## Loss curve
- Step 1: 3.908
- Step 5: 3.128
- Step 15: 2.302
- Step 30: 1.745
- Clean monotonic decrease, gradient norm stable

## Before/after on Week 1 baseline prompt
Prompt: "Bhai weekend pe Bangalore mein kya karein?"

### Qwen2.5-3B base (Week 1)
"Bhai, vartman ke shataalakshmi tithi par Bangalore mein jaane wala samay 
nahi hoga. Bangalore mein ek shataalakshmi tithi tithi hoti hai..."
[hallucinated words, made-up dates, incoherent]

### After 30 sanity steps on 100 examples
"Yaar, weekend ke liye Bangalore se jana suggestion hai! Thoda shopping 
karo, aur if you're into food, try out some local street food or a nice 
restaurant. Badiya cinema mein movie dekhna bhi baat hai..."
[natural Hinglish, useful suggestions, addresses user intent]

## Next
Day 2: real training run on full 10,594 examples, 2 epochs, with W&B logging
