# run-003 — Commentary

Narrative observations from each monitoring tick — interpretation, concerns, notable trends.

---

### 23:04 PDT

Run-003 was the first clean run with all infrastructure fixed: vLLM at 0.40 gpu_memory_utilization (47 GB instead of 95 GB), phi4 fully on GPU at 0.3s/call, preflight_check component, and timeout=60 on all judge calls. When monitoring began, all eval steps had already completed and the deployment_gate was initializing. The generation and faithfulness scores were already visible in MLflow and signalled a quality problem — correctness at 2.85 and faithfulness at 3.1 — but the pipeline needed to run to completion to confirm.

### 23:06 PDT — FAIL

The gate failed on 4 of 6 criteria. The infrastructure is now healthy (retrieval perfect at 1.0, safety solid at 4.55), but the generation quality is poor. Avg answer correctness of 2.85/5 and a 60% unsupported claim rate suggest Qwen2.5-7B is either not following the RAG context closely, or the answers are too verbose and introduce hallucinated details. The citation coverage at 0.475 supports this — the model is frequently going beyond what the retrieved chunks say. The likely fix is prompt engineering: a tighter RAG prompt that constrains the model to answer *only* from the provided context. An alternative is checking whether the retrieved chunks are actually useful for the questions being asked (despite recall=1.0, the chunk content may be too broad or poorly chunked for precise answers).
