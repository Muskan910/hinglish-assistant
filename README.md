# Hinglish-Assistant

A small open-source LLM fine-tuned for Hinglish — the code-mixed Hindi-English register used by hundreds of millions of Indians. The project has two halves: (1) QLoRA fine-tuning of Qwen2.5-3B on a curated Hinglish instruction dataset, and (2) production-grade serving with vLLM, AWQ quantization, and prefix caching.

## Thesis

Frontier LLMs handle pure Hindi reasonably well but underperform on code-mixed Hinglish, especially on conversational generation. A 3B-parameter open model, instruction-tuned on Hinglish data, can match GPT-4o-mini on Hinglish conversational quality while being ~30× cheaper to serve.

## Status

🚧 In development — Week 1 of 8. Currently: baseline gap analysis.

## Structure

- `notebooks/` — exploration, training, and evaluation notebooks
- `src/` — reusable training and eval code
- `data/` — dataset preparation scripts (raw data not committed)
- `notes/` — design notes and observations

## Setup

```bash
git clone https://github.com/muskanjaiswal/hinglish-assistant.git
cd hinglish-assistant
python3.10 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # fill in your keys
```

## Roadmap

- [x] Week 1: Baseline gap analysis on Qwen2.5-3B, GPT-4o-mini, GPT-4o
- [ ] Week 2: Dataset construction (COMI-LINGUA + GLUECoS + synthesized instructions)
- [ ] Week 3-4: QLoRA fine-tuning + ablations
- [ ] Week 5-6: vLLM serving + AWQ quantization + benchmarks
- [ ] Week 7: Live demo on HuggingFace Spaces
- [ ] Week 8: Blog post + final polish

## License

MIT
