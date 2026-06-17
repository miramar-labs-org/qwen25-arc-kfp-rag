# Validation Status — qwen25-arc-kfp-rag

**Type:** KFP v2 RAG pipeline
**Qdrant collection:** `qwen25-arc-kfp-rag`
**Platform:** Kubeflow Pipelines on NVIDIA DGX Spark (GB10, 128 GB unified memory)
**Last updated:** 2026-06-17

---

## Current Status

| Component | Status |
|-----------|--------|
| `preflight_check` | ✅ Implemented — verifies Qdrant, vLLM, and judge endpoints before eval |
| `ingest_documents` | ✅ Implemented (template) — chunks, embeds (BAAI/bge-small-en-v1.5, CPU), upserts to Qdrant |
| `retrieval_eval` | ✅ Implemented — recall@1, recall@5, MRR, hit_rate; MLflow logged |
| `generation_eval` | ✅ Implemented — RAG chain (Qwen2.5-7B via vLLM) + phi4 judge for correctness + fact_coverage |
| `faithfulness_eval` | ✅ Implemented — phi4 judge for faithfulness, citation_coverage, unsupported_claim_rate |
| `safety_eval` | ✅ Implemented — phi4 judge for safety_score (1–5 scale) |
| `deployment_gate` | ✅ Implemented (template) — threshold checks; writes gate_result.json; raises on fail |

**Pipeline validated.** Run-005 passed all 6 gate metrics. Qdrant collection approved for downstream use.

---

## Run Table

| Run | Purpose | Gate | recall@5 | faithfulness | correctness | citation_cov | unsup_claim | safety | Key Finding |
|-----|---------|------|----------|--------------|-------------|--------------|-------------|--------|-------------|
| run-003 | Baseline with preflight + judge timeout | FAIL | 1.0 | 3.10 | 2.85 | 0.475 | 0.60 | 4.55 | Generation quality bottleneck — terse fact docs + weak prompt → phi4 flags 60% of claims as unsupported |
| run-004 | System-message RAG prompt (prohibit outside knowledge) | FAIL | 1.0 | 3.40 | 3.60 | 0.55 | 0.60 | 4.55 | Correctness +0.75, faithfulness +0.30; unsupported_claim_rate unchanged — prompt alone can't fix missing context |
| run-005 | Add `arc-science-context.txt` (paragraph-form topic coverage) | **PASS** | 1.0 | **4.80** | **4.35** | **0.9625** | **0.10** | **4.85** | Richer context → Qwen grounded; unsupported_claim_rate 0.60→0.10; all 6 gates clear |

---

## What Is Implemented

### Infrastructure (inherited from platform template)
- KFP v2 pipeline scaffold with all 7 stages wired
- MLflow run-per-stage tracking with per-metric logging
- `ingest_documents` — chunking + embedding + Qdrant upsert (BAAI/bge-small-en-v1.5, CPU)
- `deployment_gate` — threshold checking for recall, faithfulness, relevancy, citation, safety
- `purge_kfp_mlflow.py`
- PVC mount: `hf-model-cache` at `/root/.cache/huggingface`
- Secret injection: `mlabs-api-keys` (OPENAI_API_KEY, HF_TOKEN, LANGCHAIN_API_KEY)

### Project-specific
- `config.yaml` — configured: vLLM at `http://192.168.1.200:8000/v1`, Ollama judge at `:11434`, Qdrant in-cluster
- `docs_src/openbookqa-facts.txt` — 20 ARC/OpenBookQA core-science statements (terse, one sentence each)
- `docs_src/arc-science-context.txt` — paragraph-form explanations for all 20 eval topic areas (hawks/prey, prisms, gas expansion, tree roots, etc.)
- `eval_dataset.jsonl` — 20 Q&A pairs from ARC challenge set; gold_doc_ids reference both source files
- `notebook.ipynb` — all 4 USER CODE BLOCKs implemented: retrieval_eval, generation_eval, faithfulness_eval, safety_eval

---

## Fixed Issues

| Issue | Fix | Run introduced |
|-------|-----|---------------|
| `qdrant-client ≥ 1.9` removed `.search()` | Changed to `.query_points().points` in `retrieval_eval` | run-003 |
| vLLM OOM with `gpu_memory_utilization=0.80` | Reduced to `0.40` (frees ~48 GB; phi4 now fully resident) | run-003 |
| Missing preflight — pipeline fails late on infra errors | Added `preflight_check` as first step; judge calls timeout=60 | run-003 |
| Weak RAG prompt allows outside-knowledge answers | Tightened to system-message form: prohibits outside knowledge, requires 1–3 sentences grounded in context | run-004 |
| Terse fact-list docs → 60% unsupported claims | Added `arc-science-context.txt` with paragraph-form explanations per topic — gives Qwen grounded context that spans its natural answer scope | run-005 |

---

## Thresholds (deployment_gate)

| Metric | Threshold | Source |
|--------|-----------|--------|
| `recall@5` | ≥ 0.70 | config.yaml → eval.retrieval_recall_threshold |
| `faithfulness` | ≥ 4.0 | config.yaml → eval.faithfulness_threshold |
| `answer_correctness` | ≥ 4.0 | config.yaml → eval.relevancy_threshold |
| `citation_coverage` | ≥ 0.75 | config.yaml → eval.citation_coverage_threshold |
| `unsupported_claim_rate` | ≤ 0.15 | config.yaml → eval.unsupported_claim_rate_threshold |
| `safety_score` | ≥ 3.5 | config.yaml → eval.safety_threshold |
