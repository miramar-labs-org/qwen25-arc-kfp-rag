# run-005 — Commentary

Narrative observations from each monitoring tick — interpretation, concerns, notable trends.

---

### 23:55 PDT — PASS

run-005 passed the deployment gate with all six metrics above threshold. The key unlock was `arc-science-context.txt`: by providing paragraph-form explanations for each topic area, the retrieved context now contains not just the core fact ("hawks eat lizards") but all the surrounding background ("hawks are birds of prey", "geckos are lizards") that Qwen naturally includes in its answers. This brought unsupported_claim_rate from 0.60 → 0.10 — from "60% of answers have unverifiable claims" to "only 2/20 do." Faithfulness jumped from 3.40 → 4.80 and citation_coverage from 0.55 → 0.9625. The `qwen25-arc-kfp-rag` Qdrant collection is now approved for downstream use. For future reference: the pattern of terse fact-list docs → hallucination is a common RAG failure mode; the fix is always to enrich context to match the natural scope of the model's output, not just to constrain the output.

### 23:49 PDT

generation_eval finished with avg_answer_correctness=4.35 and avg_fact_coverage=0.85 — a dramatic leap. Correctness has now crossed the 4.0 gate threshold for the first time (run-003: 2.85, run-004: 3.60, run-005: 4.35). The richer paragraph-form documents are giving Qwen2.5 enough grounded context to produce high-quality answers. The remaining question is faithfulness: if the richer context also eliminated the background-knowledge claims that phi4 was flagging, we should see unsupported_claim_rate drop well below 0.60. Both faithfulness_eval and safety_eval are in pip-install startup — expect active judging in under a minute. This run could PASS if the faithfulness/citation metrics follow the same upward trend.

### 23:45 PDT

Clean start: both documents ingested, retrieval perfect at 1.0 across all metrics (recall@1, recall@5, MRR, hit_rate). This confirms the new `arc-science-context.txt` was embedded and indexed alongside `openbookqa-facts.txt` without breaking retrieval. generation_eval just started its first LLM call — the key question this run is whether having paragraph-form context (rather than just terse single-sentence facts) gives Qwen2.5 enough grounded detail to write answers that phi4 can fully verify. The first vLLM response is already in flight.
