### Hi there 👋

I'm **Khoi-Nguyen Tran**, a Research Engineer with 10+ years at IBM Research (Silicon Valley Lab, San Jose, CA). I work on making LLM evaluation results trustworthy, and I build the systems that train, serve and evaluate small specialized models: adapters, quantization, local inference, and the evaluation automation that validates them.

📫 [LinkedIn](https://www.linkedin.com/in/kndtran/) · [Google Scholar](https://scholar.google.com/citations?user=ihFcT5QAAAAJ) · [Hugging Face](https://huggingface.co/kndtran)

---

## 🎯 Is the Number Real? LLM Evaluation Reliability

LLM benchmark results move for reasons that have nothing to do with the model: serving stack, batching, quantization, prompt wording, and harmless rewordings of the input. My recent work separates real gains from noise.

### perturber: a stateless perturbation layer

📄 [*perturber: A Stateless Service for Semantics-Preserving Text Perturbations*](https://arxiv.org/abs/2610.12667) (arXiv:2610.12667, 2026, sole author) · 💻 [IBM/perturber](https://github.com/IBM/perturber) (Apache-2.0)

Clean benchmarks overstate robustness, because real inputs arrive with meaning-preserving variations that routinely change model answers. Existing tools tie perturbations to a particular benchmarking stack. perturber moves them into a separate layer, decoupled from any model or evaluation harness:

- **68 perturbations across 11 families**, each applied one at a time with a seed, so results are reproducible. Callers compose perturbations by chaining calls.
- **One primitive, many front ends:** Python library, CLI, FastAPI service and Docker image, all giving the same output by construction.
- **Meaning preservation is checked, not assumed**, for every perturbation. LLM-based perturbations work with any OpenAI-compatible endpoint.

### LLM determinism

When you deploy quantized small models via Ollama for local inference, outputs vary between runs at temperature=0. Is this a fundamental GPU limitation? Does it affect downstream quality? A systematic study across **multiple model families, 4 quantization levels, 3 inference frameworks, and 2 hardware platforms** (Apple Silicon, NVIDIA H100):

- **Divergence tracks dynamic batching, not the RNG.** Variation appears under Ollama/vLLM dynamic batching, while HuggingFace Transformers is 100% deterministic on the same GPU. This is consistent with non-batch-invariant reduction kernels (the [established root cause](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/) of temperature=0 nondeterminism) rather than a CUDA or Flash Attention limitation.
- **Non-determinism is cosmetic.** 90% of mismatched outputs are semantically equivalent, benchmark accuracy (GSM8K, IFEval) shows zero variance across runs, and no individual samples flip between correct and incorrect.
- **Quantization choice matters more than hardware** for consistency. The Q4_K_M vs F16 gap is larger than the Apple Silicon vs H100 gap.
- **Prompt variation dominates** all other sources of divergence. Standardizing prompts matters more than framework or hardware choice.

## 🧩 Specialized Adapters on Small Models

RAG systems need runtime checks such as hallucination detection, answerability and citation attribution, but calling a large model for each one is slow and expensive. Small **adapters** (LoRA, aLoRA and Shadow Residuals) on a small base model do these jobs at a fraction of the cost. I built a model-agnostic harness that trains and evaluates these adapters the same way for any suitable base model, and I own training, evaluation and release readiness for Granite's RAG adapter library.

- 🤗 [ibm-granite/granitelib-rag-r1.0](https://huggingface.co/ibm-granite/granitelib-rag-r1.0): adapters extending Granite 4.x for agentic RAG (query rewrite, query clarification, context relevance, answerability, hallucination detection, citation generation). I built its **Ollama path end to end**: the GGUF conversion tooling, the Ollama launcher, and 21 GGUF adapters with Modelfiles across 4 base models.
- 🤗 [ibm-granite/granitelib-rag-gpt-oss-r1.0](https://huggingface.co/ibm-granite/granitelib-rag-gpt-oss-r1.0): an experimental adapter set for `gpt-oss-20b`.
- **Serving it:** one Ollama instance serves 6+ RAG operations from a single small base model, switching adapters per request with no model reload and no GPU cluster. I shipped the production API (FastAPI + OpenShift), a CLI for training and uploading custom adapters, and a visual pipeline builder (custom Langflow components, ChromaDB/ELSER retrieval, Langfuse tracing) that 20+ researchers across teams use to build and compare flows.

### Adapters vs. decision models

Decision models return a probability for each option of a question instead of generating text, which makes them a natural general-purpose alternative to task-specific adapters. I added decision-model evaluation to our adapter harness, mapping each RAG task to a decision request so that both are scored with the same data and metrics. I then compared an open-weight decision model and three hosted decision APIs against the Granite 4.1 adapters. The results are published on the adapter [model cards](https://huggingface.co/ibm-granite/granitelib-rag-r1.0) and in the [granitelib-evals Space](https://huggingface.co/spaces/ibm-granite/granitelib-evals). They back the [Granite Switch 4.1 launch](https://huggingface.co/blog/ibm-granite/granite-switch-4-1) claim that specialized adapters can beat generalist decision models.

| Task | Granite 4.1 adapters (3B–30B) | Decision models | Prompted Granite 4.1 |
|---|---|---|---|
| Query clarification (accuracy) | 94.4–96.1% | 84.1–86.4% | 47.5–76.2% |
| Answerability (accuracy) | 89.4–92.2% | 85.2–91.2% | 59.4–65.5% |
| Hallucination detection (macro F1) | 70.8–82.9 | 54.8–73.1 | 41.8–53.6 |

- **Small adapters hold their own.** Even the 3B adapters lead clearly on query clarification and match the best decision model on answerability. On hallucination detection, the 30B adapter leads by about 10 points.
- **Both approaches are far ahead of prompting** the same Granite models.
- **Caveat:** the decision models weren't adapted to these tasks beyond the wording of each question, so their scores depend on that wording.
- **Next:** how decision models respond to meaning-preserving input perturbations (with perturber) and to serving noise.

📄 Co-author: [*A Library of LLM Intrinsics for Retrieval-Augmented Generation*](https://arxiv.org/abs/2504.11704) (arXiv:2504.11704, 2025).

## 🔌 Agent Infrastructure

**MCP search router:** agents need one search tool that can reach both the open web and private document collections. I designed, built and deployed a router that sends each query to the right backend: web search, Elasticsearch semantic search, or Qdrant vector search with neighboring-passage context.
- **One core, three interfaces:** the same routing code serves an MCP tool, a REST API and a web UI.
- **Ingestion at scale:** an offline pipeline with pluggable chunkers, streaming datasets and sharded parallel embedding, run as Kubernetes jobs, that loads agentic-search benchmark corpora such as BrowseComp-Plus.
- **Runs in a locked-down cluster:** containerized on OpenShift with no outbound internet access, with the embedding model built into the image.
- **Adopted by other research projects:** two agent projects use it. For one of them, I contributed an MCP client that reads each server's tool schema, so a single tool works with any MCP server.

## 📊 Benchmarking & Evaluation Automation

- **RAG benchmarks for the Granite 4.0 launch:** I produced the RAG benchmark results featured in IBM's [Granite 4.0 announcement](https://www.ibm.com/new/announcements/ibm-granite-4-0-hyper-efficient-high-performance-hybrid-models), with a harness I built that covers RAGBench, MTRAG and other RAG datasets across vLLM, Ollama and hosted endpoints.
- **Quantization quality:** Granite 4.0 Micro across quantization levels (Q4_K_M to F16) on GSM8K and IFEval. Q4_K_M runs at 76 tok/s with competitive accuracy; F16 is roughly 1.8x slower. Reasoning quality degrades non-uniformly across tasks.
- **Adapter conversion:** validated that adapters produce equivalent results served through Ollama, vLLM and direct HuggingFace inference.
- **RAG patterns:** evaluated composable adapter patterns (query rewrite plus retrieval, hallucination-feedback loops) against MTRAG, measuring faithfulness, correctness and relevancy.
- **Resource & cross-hardware profiling:** RAM/VRAM usage and KV-cache scaling per quantization level; validated that M1 Mac dev benchmarks transfer to H100 production hardware.

## 🌐 Open Source Contributions

These public PRs are the visible tip of a much larger body of **inner-source** work at IBM, most of which lives behind the firewall. What's public sits in IBM's Granite adapter stack: the consuming library was redeveloped from **granite-io** to **granite-common**, then integrated into **mellea**, and I contributed across that full evolution.

- **[ibm-granite/granite-io](https://github.com/ibm-granite/granite-io)** (original library): Elasticsearch retrieval support, composite pipeline fixes, and watsonx/litellm integration fixes (7 merged PRs).
- **[ibm-granite/granite-common](https://github.com/ibm-granite/granite-common)** (the redevelopment): the retrievers module, Ollama backend support, structured-output spec updates, output sanitization, query-clarification tests, and CI/test infrastructure (13 merged PRs).
- **[generative-computing/mellea](https://github.com/generative-computing/mellea)** (the integration): Ollama model-name mappings for Granite 4.1 adapter resolution ([#1085](https://github.com/generative-computing/mellea/pull/1085)).
- **[huggingface/transformers](https://github.com/huggingface/transformers)**: identified and fixed a silent v4→v5 tokenizer regression in which `AutoTokenizer` produced incorrect token IDs by applying the GPT-2 pre-tokenizer rather than the model's own `tokenizer.json`.
  - Traced the defect to the `AutoTokenizer` routing logic, then surveyed ~1,700 tokenizers from the most-downloaded Hub models: 62 were affected, about 30 of them still broken (over 3M combined downloads).
  - Contributed the [merged fix for Granite](https://github.com/huggingface/transformers/pull/45813) ([issue](https://github.com/huggingface/transformers/issues/45812)), shipped in v5.9.0, and filed a [follow-up report](https://github.com/huggingface/transformers/issues/45920) that led maintainers to a broader fix.
- **Upstream bug reports:** [vllm-project/vllm#27825](https://github.com/vllm-project/vllm/issues/27825), a LoRA conversion bug in llama.cpp (fixed upstream in [#22832](https://github.com/ggml-org/llama.cpp/pull/22832), where I was consulted), and a Metal F16 overflow crash in Granite 1B GGUF on Apple Silicon.

## 🛠️ How I Work

I've spent my career building across the entire vertical: API servers, visual pipeline builders, cloud deployment, observability, benchmarking harnesses, and I still own all of it. What's changed is that **Claude Code** has made the coding itself dramatically more efficient, so I spend less time typing implementations and more on the technical designs and architecture that actually drive the work. The hard part is the thinking: breaking an ambiguous research idea into a well-structured, well-architected plan, and judging whether to build at all. Reviewing and correcting a coding agent, done well, is dramatically faster than typing the code yourself. I've taught this research-to-production workflow to researchers and engineers across several teams.

## 📚 Publications & Background

- **Ph.D., Computer Science**, The Australian National University · IEEE Senior Member
- **640+ citations · h-index 13** ([Google Scholar](https://scholar.google.com/citations?user=ihFcT5QAAAAJ)) · 12 US patents and applications, plus 1 filed in 2026 · IBM TCAP Award (2024)
- **Selected papers:** [perturber](https://arxiv.org/abs/2610.12667) (arXiv 2026, sole author) · [A Library of LLM Intrinsics for RAG](https://arxiv.org/abs/2504.11704) (arXiv 2025) · [PriMeSRL-Eval](https://arxiv.org/abs/2210.06408) (Findings of EACL 2023) · [Universal Proposition Bank 2.0](https://aclanthology.org/2022.lrec-1.181/) (LREC 2022); earlier work at WWW, CIKM, IEEE TKDE, IJCNLP, PAKDD and AAAI
- **Earlier at IBM:** led delivery of the first semantic role labeling models to Watson NLP, IBM's inner-source NLP library used in 15+ products
