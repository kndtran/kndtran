### Hi there 👋

I'm **Khoi-Nguyen Tran**, a Research Engineer at IBM Research (San Jose, CA). I make LLM evaluation results trustworthy, and I build the systems that train, serve and evaluate small specialized models.

📫 [LinkedIn](https://www.linkedin.com/in/kndtran/) · [Google Scholar](https://scholar.google.com/citations?user=ihFcT5QAAAAJ) · [Hugging Face](https://huggingface.co/kndtran)

---

## 🎯 LLM Evaluation

- **Granite RAG evaluation:** I produced the RAG benchmark results for IBM Granite models from 3.x through 4.1, including those in the [Granite 4.0 launch](https://www.ibm.com/new/announcements/ibm-granite-4-0-hyper-efficient-high-performance-hybrid-models), using a harness I built (MTRAG, RAGBench and more, across vLLM, Ollama and hosted endpoints).
- **[perturber](https://github.com/IBM/perturber)** ([paper](https://arxiv.org/abs/2610.12667), sole author): a stateless perturbation layer, decoupled from any model or benchmark stack. It applies 68 seeded, meaning-preserving perturbations, one call at a time, so robustness results are reproducible.
- **Determinism:** temperature-0 differences come from dynamic batching in Ollama and vLLM, not the GPU. HF Transformers is 100% deterministic on the same hardware, and the differences don't change benchmark accuracy.
- **transformers v5 bug:** found and [fixed](https://github.com/huggingface/transformers/pull/45813) a silent tokenizer regression that gave wrong token IDs for every Granite model. I [audited](https://github.com/huggingface/transformers/issues/45920) ~1,700 Hub tokenizers and found 62 affected.

## 🧩 Specialized Adapters on Small Models

Small adapters (LoRA, aLoRA, Shadow Residuals) handle RAG checks like hallucination detection and answerability at a fraction of a large model's cost. I built a model-agnostic harness that trains and evaluates them, and I own release readiness for [Granite's RAG adapter library](https://huggingface.co/ibm-granite/granitelib-rag-r1.0), including its full Ollama/GGUF path.

**Adapters vs. decision models:** I produced the comparison behind the [Granite Switch 4.1 launch](https://huggingface.co/blog/ibm-granite/granite-switch-4-1) ([Space](https://huggingface.co/spaces/ibm-granite/granitelib-evals)):

| Task | Granite 4.1 adapters | Decision models | Prompted Granite 4.1 |
|---|---|---|---|
| Query clarification | 94.4–96.1% | 84.1–86.4% | 47.5–76.2% |
| Answerability | 89.4–92.2% | 85.2–91.2% | 59.4–65.5% |
| Hallucination detection (F1) | 70.8–82.9% | 54.8–73.1% | 41.8–53.6% |

## 🏗️ Systems

- **Serving:** one Ollama instance serves 6+ RAG adapters per request from a single small model, behind a FastAPI service on OpenShift and a Langflow pipeline builder used by 20+ researchers.
- **MCP search router:** one routing core exposed as an MCP tool, a REST API and a web UI over web, Elasticsearch and Qdrant search. Two agent projects use it.

## 🌐 Open Source

- **Granite libraries:** [granite-io](https://github.com/ibm-granite/granite-io) → [granite-common](https://github.com/ibm-granite/granite-common) → [mellea](https://github.com/generative-computing/mellea), about 20 merged PRs, including the retrievers module and the Ollama backend.
- **Upstream reports:** [huggingface/transformers](https://github.com/huggingface/transformers/issues/45812), [vllm#27825](https://github.com/vllm-project/vllm/issues/27825), and a llama.cpp LoRA bug ([fixed upstream](https://github.com/ggml-org/llama.cpp/pull/22832)).

## 🛠️ How I Work

I build across the whole stack, from research to production, with **Claude Code**. The hard part is the thinking: turning an ambiguous research idea into a well-architected plan, and judging whether to build at all. Agents do most of the typing, and I review every change. I've taught this workflow to researchers and engineers across several teams.

## 📚 Background

Ph.D., The Australian National University · IEEE Senior Member · 640+ citations, h-index 13 ([Scholar](https://scholar.google.com/citations?user=ihFcT5QAAAAJ)) · 12 US patents and applications · IBM TCAP Award (2024)

Selected papers: [perturber](https://arxiv.org/abs/2610.12667) (2026) · [A Library of LLM Intrinsics for RAG](https://arxiv.org/abs/2504.11704) (2025) · [PriMeSRL-Eval](https://arxiv.org/abs/2210.06408) (Findings of EACL 2023) · [Universal Proposition Bank 2.0](https://aclanthology.org/2022.lrec-1.181/) (LREC 2022)
