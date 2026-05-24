# Evaluation dataset

`eval_set.jsonl` — one line per case: `{ "question": {...}, "ground_truth_verdict": {...} }`.

- `question` conforms to `schema/question.schema.json`.
- `ground_truth_verdict` conforms to `schema/verdict.schema.json` and is **your human label** (the truth you measure the model against).

## Your task (Week 1 · Session 1): reach 20–30 cases

Today there are 3 example cases (one PASS, one critical FAIL, one quality REVIEW). Grow the set to **20–30** with this suggested distribution, so every rule has signal:

| Type | Approx. count | Covers |
|------|---------------|--------|
| Clean PASS | ~10 | The validator must not reject good questions (avoid false positives) |
| FAIL R1 (malformed) | ~2 | Missing option / two correct marked / no rationale |
| FAIL R2 (two defensible answers) | ~3 | The classic: two correct options |
| FAIL R3 (not anchored to objective) | ~2 | Question is about a different topic than the declared objective |
| FAIL R4 (leakage / protected material) | ~2 | Text that mimics a real exam question (seeded, clearly marked) |
| REVIEW R5 (absurd distractors) | ~2 | Filler distractors, nobody would pick them |
| REVIEW R6 (wording) | ~2 | "All of the above", confusing negations, grammatical tells |

Balance matters: if everything is PASS, you learn nothing about failures; if everything is FAIL, you don't detect false positives.

## Golden rules

- **100% original material** anchored to **public** official objectives. Never copy real exam questions or protected vendor text. The leakage (R4) cases you seed must be invented by you to *look like* leakage, not real leaks.
- Label yourself first, without peeking at what the model would say. Ground truth is your human judgment.
- A question can fail multiple rules; label all of them. `overall` follows the logic in `schema/rules.json` (FAIL if any critical fails; REVIEW if only quality rules fail).
- When you doubt your own label, that question is gold: note it, edge cases teach the most.

## Validating the format

Before running the harness it's a good idea to check that each line parses and matches the schema. We'll do that in Session 2 alongside the evaluator.
