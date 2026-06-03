## Final dataset: hinglish_synthetic_v1_clean.jsonl

### Stats
- Total: 10,594 examples (after filtering 10,839 raw)
- Survival rate: 97.7%
- Filter removals: 132 exact duplicates, 111 near-duplicates, 2 format pollution
- Categories: casual (3391), customer_support (2880), question (2847), sentiment (1476)
- Script: 100% Roman (mixed-script reserved for v2)

### Quality assessment
Random 30-sample spot-check (seed=42): 27-28 of 30 training-worthy.
Minor issues observed:
- Occasional fabricated personal anecdotes (~1 in 30)
- Slight over-affirmation in advice scenarios (~1 in 30)
Major failure modes (AI disclaimers, English drift, markdown leaks): 0 of 30.

### Methodology
- Generator: GPT-4o-mini via OpenAI API, response_format=json_object
- Diversity sampling: 20 personas × 90 scenarios × 12 styles (~21,600 combinations)
  → expected duplicate rate ~2%, observed 2.2%, matches
- Synthesis prompt: 615 tokens, 8 negative-constraint rules
- Parallelization: ThreadPoolExecutor concurrency=10, ~120 examples/min
- Total cost: ~₹400 (Week 2)

### Known limitations (acknowledged for blog post)
1. 100% Roman script — does not address mixed-script Hinglish (deferred to v2)
2. Generator and one of the eval targets (GPT-4o-mini) share a vendor — mitigated 
   by using Claude as LLM-as-judge in Week 4
3. Occasional fabricated personal anecdotes — acceptable since assistant is 
   personified as a "human friend"