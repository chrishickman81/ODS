# Local model comparison history

A running baseline of how candidate local models compare with the production model on a fixed set of
Salesforce developer tasks (Apex, LWC/Jest, metadata, SOQL, analysis). Add one section per round and leave
old rounds unchanged, so later changes to models, llama.cpp or tooling can be compared against them.
Raw task inputs, model outputs and org-specific oracle details are kept outside the repository.

## Method
- Fixed task set: 24 "hard" tasks (never-passed, flaky, or weak-check in an earlier eval) plus 6 easy controls; 2 runs per task per model.
- Same settings for every model: temperature 0.2, 16384 max tokens, same system prompt, reasoning off (gpt-oss: effort low or medium), `--parallel 1`, 131072 context.
- Scoring uses automatic oracles per task: static checks, unit tests with mutants, Jest, and Salesforce check-only validates plus read-only queries (no deploys, no record writes).
- Always re-run the current production model as the control arm: results shift with llama.cpp, the oracles and the org.
- Offline-only scoring is too lenient; include the org-backed oracles.

## Round 1: 2026-11-20
Hardware: Mac Studio M4 Max, 128 GB. llama.cpp b11404.

| Model (speculative decoding) | Hard runs passed | Hard tasks passing at least once | Control runs passed | Median s/run | Median tok/s |
|---|---|---|---|---|---|
| Qwen3.6-35B-A3B UD-Q6_K_XL, MTP + ngram-mod (production) | 9/48 | 7/24 | 12/12 | 15 | 138 |
| gpt-oss-120b MXFP4, ngram-mod, effort low | 13/48 | 9/24 | 5/12 | 20 | 85 |
| gpt-oss-120b MXFP4, ngram-mod, effort medium | 15/48 | 11/24 | 8/12 | 40 | 84 |
| Qwen3-Coder-Next UD-Q4_K_XL, ngram-mod | 10/48 | 7/24 | 12/12 | 25 | 72 |

Findings
- Hard-task differences between models are within noise at 2 runs per task.
- gpt-oss-120b was ahead only on SOQL generation and one metadata-conversion task, at both effort levels. It failed easy control tasks and is 1.3 to 2.7 times slower.
- All models failed the Jest and Apex-test generation tasks and several analysis tasks.
- Decision: keep the Qwen3.6 Q6 + MTP model; no switch.
