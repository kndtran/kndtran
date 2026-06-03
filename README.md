### Hi there 👋

I'm **Khoi-Nguyen Tran**, a Research Engineer with 10+ years at IBM Research (Silicon Valley Lab, San Jose, CA). I build small-model LLM systems for RAG, covering adapters, quantization, and local inference, together with the evaluation research and automation that validates them.

📫 [LinkedIn](https://www.linkedin.com/in/kndtran/) · [Website](https://kndtran.github.io/)

---

## 🧩 LLM Intrinsics: Hot-Swappable LoRA Adapters on Small Models

RAG systems need runtime guardrails such as hallucination detection, answerability scoring, and citation attribution, but calling a large model for each check is too slow and expensive. I built an intrinsics framework that serves **6+ specialized RAG operations from a single Granite Micro base model** using LoRA/aLoRA adapters, switching adapters per request through one Ollama instance, with no model reload and no GPU cluster. I shipped the production API (FastAPI + OpenShift), a CLI for training and uploading custom adapters, and a visual pipeline builder (custom Langflow components, ChromaDB/ELSER retrieval, Langfuse tracing) so researchers can drag-and-drop intrinsic nodes and compare backends side by side.

The intrinsics themselves are published as LoRA adapters on Hugging Face, where I contributed the **Ollama GGUF conversions**. The granite-common and mellea libraries (below) consume these repos:

- 🤗 [ibm-granite/granitelib-rag-r1.0](https://huggingface.co/ibm-granite/granitelib-rag-r1.0): six LoRA/aLoRA adapters extending Granite 4.0/4.1 for agentic RAG (query rewrite, query clarification, context relevance, answerability, hallucination detection, citation generation).
- 🤗 [ibm-granite/granitelib-rag-gpt-oss-r1.0](https://huggingface.co/ibm-granite/granitelib-rag-gpt-oss-r1.0): experimental adapter set for `gpt-oss-20b`.

📄 Co-author: [*A Library of LLM Intrinsics for Retrieval-Augmented Generation*](https://arxiv.org/abs/2504.11704) (arXiv:2504.11704, 2025).

## 🔬 LLM Determinism Research

When you deploy quantized small models via Ollama for local inference, outputs vary between runs at temperature=0. Is this a fundamental GPU limitation? Does it affect downstream quality? Can we trust quantized models for reproducible evaluation? A systematic study across **5 model families, 4 quantization levels, 3 inference frameworks, and 2 hardware platforms** (Apple Silicon, NVIDIA H100):

- **Divergence tracks dynamic batching, not the RNG.** Variation appears under Ollama/vLLM dynamic batching while HuggingFace Transformers is 100% deterministic on the same GPU. This is consistent with non-batch-invariant reduction kernels (the [established root cause](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/) of temperature=0 nondeterminism) rather than a CUDA or Flash Attention limitation.
- **Non-determinism is cosmetic.** 90% of mismatched outputs are semantically equivalent, benchmark accuracy (GSM8K, IFEval) shows zero variance across runs, and no individual samples flip between correct and incorrect.
- **Quantization choice matters more than hardware** for consistency. The Q4_K_M vs F16 gap is larger than the Apple Silicon vs H100 gap.
- **Prompt variation dominates** all other sources of divergence. Standardizing prompts matters more than framework or hardware choice.
- Discovered and reported a **Metal F16 overflow crash** in Granite 1B GGUF on Apple Silicon (filed upstream).

## 📊 Benchmarking & Evaluation Automation

Beyond determinism, I built the benchmarking automation to answer the practical deployment questions: how do GGUF quantization levels trade quality for speed, do converted LoRA adapters hold up, and which compositions of intrinsics improve RAG?

- **Quantization quality:** Granite 4.0 Micro across quantization levels (Q4_K_M to F16) on GSM8K and IFEval. Q4_K_M runs at 76 tok/s with competitive accuracy; F16 is roughly 1.8x slower. Reasoning quality degrades non-uniformly across tasks.
- **Adapter conversion:** Validated that intrinsic operations produce equivalent results served through Ollama vs vLLM vs direct HuggingFace inference.
- **RAG patterns:** Evaluated composable intrinsic patterns (query-rewrite plus retrieval, hallucination-feedback loops) against MTRAG, measuring faithfulness, correctness, and relevancy.
- **Resource & cross-hardware profiling:** RAM/VRAM usage and KV-cache scaling per quantization level; validated that M1 Mac dev benchmarks transfer to H100 production hardware.

## 🌐 Open Source Contributions

These public PRs are the visible tip of a much larger body of **inner-source** work at IBM, including a full evaluation automation framework, most of which lives behind the firewall. What's public sits in IBM's Granite intrinsics stack: the consuming library was redeveloped from **granite-io** to **granite-common**, then integrated into **mellea** (these consume the Hugging Face adapter repos above), and I contributed across that full evolution.

- **[ibm-granite/granite-io](https://github.com/ibm-granite/granite-io)** (original library): Elasticsearch retrieval support, composite-intrinsics fixes, and watsonx/litellm integration fixes (7 merged PRs).
- **[ibm-granite/granite-common](https://github.com/ibm-granite/granite-common)** (the redevelopment): Ollama backend support, structured-output spec updates, output sanitization, query-clarification tests, and CI/test infrastructure (12 merged PRs).
- **[generative-computing/mellea](https://github.com/generative-computing/mellea)** (the integration): Ollama model-name mappings for Granite 4.1 intrinsics adapter resolution.
- **[huggingface/transformers](https://github.com/huggingface/transformers)**: identified and fixed a silent v4→v5 tokenizer regression in which `AutoTokenizer` produced incorrect token IDs by applying the GPT-2 pre-tokenizer rather than the model's own `tokenizer.json`.
  - Traced the defect to the `AutoTokenizer` routing logic, then surveyed the HuggingFace top-1000 models (~1700 tokenizers) to assess its scope: 62 affected model families representing over 3M combined downloads.
  - Contributed the [merged fix for Granite](https://github.com/huggingface/transformers/pull/45813) ([issue](https://github.com/huggingface/transformers/issues/45812)) and filed a [follow-up report](https://github.com/huggingface/transformers/issues/45920) extending the analysis to OLMo2, HyperClovaX, DeepSeek-R1-Distill-Llama, Yi, and additional model families.

## 🛠️ How I Work

I've spent my career building across the entire vertical: API servers, visual pipeline builders, cloud deployment, observability, benchmarking harnesses, and I still own all of it. What's changed is that **Claude Code** has made the coding itself dramatically more efficient, so I spend less time typing implementations and more on the technical designs and architecture that actually drive the work. It's a multiplier on productivity: projects that used to take weeks now ship in days. The hard part is the thinking, the design, the planning, and the judgment of whether to build at all. The rest is a skill: reviewing and correcting a coding agent, done correctly, is dramatically faster than typing the code yourself.

## 📚 Prior Work

Before LLMs, I worked on semantic role labeling (SRL) and NLP systems, delivering models to IBM Watson/watsonx products.

- 20+ peer-reviewed publications · 12 patents · 4 IBM technical awards
- IEEE Senior Member · Ph.D., The Australian National University
- [Universal Propositions 2.0](https://universalpropositions.github.io/) ([paper](https://aclanthology.org/2022.lrec-1.181/)): Vietnamese SRL model and BERT fine-tuning for Watson NLP
- [PriMeSRL-Eval](https://github.com/UniversalPropositions/PriMeSRL-Eval) ([paper](https://arxiv.org/abs/2210.06408)): open-source SRL evaluation framework
