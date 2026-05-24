# Project 1 — Prompt Lab & Evaluations

> ⏱ 2 weeks (~6 h) · 🔗 Foundation for every later project · Status: **v0.1 (scope defined, harness not built yet)**

## What this is, in one sentence

A lab for comparing prompt strategies **with metrics** on a real task from the cert engine —**validating/auditing practice questions**— and also my harness for model comparison and migration. The goal isn't a perfect prompt: it's the habit of **measuring instead of guessing**.

## The problem (scoped)

**Task:** given a multiple-choice practice question and the official objective it claims to belong to, emit a **structured verdict**: is it valid? Which rules does it pass and which does it fail?

**Why this task and not another:**
- It has **clean ground truth** (PASS/FAIL labels per rule) → crisp metrics from v0.1, no dependence on a subjective judge.
- It's the **legal/ethical guardrail** of the project: the "no real exam leakage or protected material" rule becomes an automatable test, not just good intentions.
- It's the **most reusable piece** of the engine: it comes back as a *quality gate* for content generation in the capstone, as an eval in other projects, and as a skill in Project 2.

**Cert-agnostic by design:** the certification and objective enter as **input** (`cert_id`, `objective_id`). The validator knows nothing about Security+ specifically; tomorrow you pass it a cloud cert objective and it works the same. Nothing is hardcoded to any one exam.

### In scope (v0.1)
- Input: **one** question (JSON) + its declared objective.
- Output: per-rule structured verdict + overall verdict (see [data contract](#data-contract)).
- Compare **3–4 prompt versions** on the same dataset.
- Dataset of **20–30 labeled questions** (mix of clean PASS and seeded FAILs covering every rule).
- Script-computed metrics: verdict accuracy, per-rule accuracy, % valid format, cost, latency.

### Out of scope (v0.1) — to avoid scope creep
- **Generating** questions (that's another task/tool; here we only validate).
- DB and UI (we use JSONL + CSV).
- Multi-cert dataset (the schema supports it, but v0.1 starts with one cert).
- LLM-as-judge and automatic multi-model routing → see [Stretch challenges](#stretch-challenges).

## Stack

- **Language:** TypeScript (where I'm most fluent). The harness lives in `src/` (Node).
- **SDK:** Anthropic TypeScript SDK (`@anthropic-ai/sdk`) for the calls and token counting; structured output validated against `verdict.schema.json`.
- **Storage:** JSONL for raw results in `results/` + CSV summary. No DB yet.
- **Model:** *none fixed*. Picked fresh at run time (Week 1·Session 2), never from memory — see [Model policy and migration rule](#model-policy-and-migration-rule-written).

> Portfolio note: language is chosen per project. **Project 5 (RAG)** is best done in **Python** because the embeddings and vector store ecosystem is more mature there; there's no obligation for the whole monorepo to be TS.

## The validation rules (the rubric)

The validator evaluates each question against these rules. **R1–R4 are critical** (if they fail, verdict is FAIL). **R5–R6 are quality** (they produce REVIEW/warning, not an automatic FAIL).

| Rule | Critical | What it checks |
|------|----------|----------------|
| R1 Well-formed | ✅ | Has a stem, ≥3 options, **exactly one** marked correct, and a rationale. |
| R2 Single defensible answer | ✅ | Only one option is correct; the others are unambiguously incorrect. |
| R3 Anchored to objective | ✅ | The question evaluates the declared `objective_id`, not another topic. |
| R4 No leakage / protected material | ✅ | Does not reproduce real questions or protected vendor text (compliance). |
| R5 Plausible distractors | ⬜ | Distractors are believable to someone who hasn't mastered the topic, not absurd. |
| R6 Wording quality | ⬜ | No confusing negations, no "all/none of the above" as a crutch, no grammatical tells that give away the answer, no ambiguity. |

> The rule catalog lives in `data/schema/` as a versioned contract. If you change a rule, that's a contract change: note it.

## Data contract

- **Input** — `data/schema/question.schema.json`: a practice question (cert/objective as fields).
- **Output** — `data/schema/verdict.schema.json`: the verdict (per rule + overall + reasons).
- **Dataset** — `data/eval_set.jsonl`: one line per case = `{ question, ground_truth_verdict }`.

Defining the contract before the code is deliberate: it forces the lab to be a box with a clean interface (question in, verdict out), reusable and testable.

## Metrics

| Metric | Definition | Role |
|--------|------------|------|
| **Verdict accuracy** | % of cases where the overall PASS/FAIL/REVIEW matches ground truth | **Primary** |
| Per-rule accuracy | % correct for each R1–R6 (helps see *where* the prompt fails) | Diagnostic |
| % valid format | % of outputs that parse against `verdict.schema.json` | Contract health |
| Cost per response | USD per verdict (in/out tokens × model price) | Model decision |
| Latency | p50 / p95 per verdict | Model decision |

**Improvement criterion (from the spec):** the best prompt must beat the basic version by **≥20%** on the primary metric, without regressing the critical rules.

## Model policy and migration rule (WRITTEN)

Model names and prices change: **fetch fresh at run time (Week 1·Session 2), never from memory.** This lab is the comparison harness: same winning prompt + same dataset, multiple models, decision by the numbers.

There is a **reference model (champion)** —the one currently in use— and **challenger models**. A challenger replaces the champion **only if**, on the same dataset and the same winning prompt, it meets **all** of these conditions:

1. **No regression** in verdict accuracy (ties allowed).
2. Delivers one of: **+3 pp** on the primary metric, **or** parity on accuracy with **≥30% lower cost**/response, **or** parity on accuracy with **≥30% lower p50 latency**.
3. **% valid format ≥ 98%** (doesn't break the JSON contract).
4. **No regression > 2 pp** on any critical rule (R1–R4).

If a challenger wins, the date, prompt commit, and metrics table are recorded in `results/` **before** migrating. The decision is never made by eye. *(v0.1 thresholds, revisable — but the change stays written.)*

## Work plan

1. **Week 1 · Session 1** — *(this checkpoint)* Define rules and contract ✅. Collect **20 labeled questions** (mix of PASS/FAIL covering every rule) in `data/eval_set.jsonl`.
2. **Week 1 · Session 2** — Write 3 prompt versions in `prompts/`: `v1_zero_shot`, `v2_few_shot`, `v3_cot`, all with structured output. Look up **fresh** models and pick the comparison set. Run over the dataset; save raw results in `results/`.
3. **Week 2 · Session 1** — Build the evaluator (`src/`): verdict accuracy + per-rule accuracy, % valid format, cost, latency. Comparison table.
4. **Week 2 · Session 2** — Analyze the 5 worst failures of the best prompt. Create `v4_targeted` aimed at those cases. Measure that it improves **without breaking** the rest. Write up conclusions + thread/post.

## Deliverables

- Versioned prompts (`prompts/`), evaluation dataset (`data/`), run script and metrics script (`src/`).
- This README with a **comparison table of the 4 versions** and conclusions.
- A **"winning prompt"** with its baseline metric — reused in future projects.

## Success criteria

- I can justify why one prompt is better than another **with data, not intuition**.
- The best prompt beats the basic version by **≥20%** on the primary metric.
- Repeatable process: when I need a new prompt tomorrow, I know how to attack it methodically.

## Stretch challenges (expert level)

- Use a smaller model as a **judge** (LLM-as-judge) for the quality rules R5–R6.
- **Automatic routing**: cheap model for easy cases, powerful one for hard cases.
- **Prompt caching** and measuring the cost/latency savings.

## Legal/ethical note

All questions in the dataset are **original material** anchored to public official objectives. **Never** are real exam questions or protected material from CompTIA or other vendors included. Rule R4 exists precisely to detect and reject leakage.
