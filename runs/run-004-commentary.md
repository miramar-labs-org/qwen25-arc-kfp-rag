# run-004 — Commentary

Narrative observations from each monitoring tick — interpretation, concerns, notable trends.

---

### 23:29 PDT — FAIL

run-004 failed the deployment gate. The system-message RAG prompt showed clear improvement in correctness (+0.75: 2.85→3.6) and modest gains in faithfulness (+0.3: 3.10→3.4) and citation_coverage (+0.075: 0.475→0.55). However, the most stubborn metric — unsupported_claim_rate — stayed exactly at 0.60 in both run-003 and run-004. This locked value is a signal: the same 12 out of 20 questions have at least one claim the phi4 judge deems unsupported by the retrieved context, and the system message didn't break that pattern. The root cause is likely structural: ARC-Challenge questions require specific domain facts that aren't fully present in the ingested openbookqa-facts document, so Qwen2.5 has to reach outside the context regardless of how tightly prompted. The choices are (1) richer documents that cover the question set, (2) a more lenient judge prompt that allows inference from context rather than requiring verbatim support, or (3) threshold adjustment if the dataset/model combination genuinely can't meet 0.15 unsupported rate.

### 23:28 PDT

generation_eval finished all 20 rows: avg_answer_correctness=3.6, avg_fact_coverage=0.675. The tightened system-message prompt meaningfully improved correctness (+0.75 over run-003's 2.85), but 3.6 is still below the 4.0 gate threshold. Faithfulness and safety are now running in parallel — faithfulness_eval is actively judging at GPU speed (~2s/call) and should finish the 20 rows in under a minute; safety_eval is in pip-install startup. If the context constraint also reduced hallucinations, we'd expect faithfulness and citation_coverage to improve over run-003 (3.10 and 0.475 respectively), potentially clearing those gates. The make-or-break question for PASS is whether faithfulness_eval clears 4.0 and citation_coverage reaches 0.75 — correctness at 3.6 will still FAIL that gate regardless.

### 23:20 PDT

The tighter system-message RAG prompt is showing immediate impact. The first graded question scored 5.0 correctness and 1.0 fact coverage — a perfect score — compared to the 2.85 average seen across all 20 questions in run-003. The judge is also responding in ~2-9 seconds per row (fully on GPU), vs the timeout-level latency that plagued run-002. retrieval_eval is again perfect at 1.0 across all metrics. The key question is whether the improvement holds across the remaining ~18 questions, particularly the ones that scored low in run-003 (the "Shining light through a diamond" question already shows corr=3.0 on row 2, so variance is still present).

### 23:14 PDT

run-004 started cleanly — preflight_check completed without errors, confirming Qdrant, vLLM, and Ollama/phi4 are all reachable before any heavy eval work. The `ingest_documents` step is now running, embedding the openbookqa-facts document into Qdrant. This is the first run with the tightened system-message RAG prompt; the key question is whether constraining Qwen2.5 to context-only answers will reduce the 60% unsupported claim rate seen in run-003.
