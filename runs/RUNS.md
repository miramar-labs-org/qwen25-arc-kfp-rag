# RUNS

| Run | Date | Gate | Retrieval R@5 | Faithfulness | Relevancy | Citation Cov | Unsupported Rate | Safety | Notes |
|---|---|---|---|---|---|---|---|---|---|
| run-001 | 2026-06-16 | FAIL | — | — | — | — | — | — | qdrant .search() AttributeError |
| run-002 | 2026-06-16 | FAIL | — | — | — | — | — | — | Terminated — phi4 running on CPU (vLLM consumed 95 GB), judge timeouts |
| run-003 | 2026-06-17 | FAIL | 1.0 | 3.10 | 2.85 | 0.475 | 0.60 | 4.55 | All infra fixed; retrieval perfect; generation/faithfulness quality below thresholds |
| run-004 | 2026-06-17 | FAIL | 1.0 | 3.40 | 3.60 | 0.55 | 0.60 | 4.55 | Tightened RAG prompt (system message); correctness/faithfulness improved; unsupported_claim_rate stuck at 0.60 |
