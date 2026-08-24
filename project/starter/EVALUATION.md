# Evaluation Observations

Written observations for the **Testing and Evaluation** rubric criterion.

## What was tested

The chatbot (AgentCore managed harness, model `us.amazon.nova-pro-v1:0`) was
tested with the suite in [`harness-tests.json`](./harness-tests.json) — 11 cases
covering all three routes plus edge cases:

| Route | Test IDs | What they check |
|-------|----------|-----------------|
| Bug report | `t1`, `t2`, `t3`, `t4` | One-field-at-a-time collection; all-info-in-one-turn tool call; vague start; bug-vs-FAQ compound message |
| Platform question | `t5`, `t6`, `t7` | Covered question (return policy), paraphrased covered question, uncovered question (price matching → support line) |
| Other | `t8` | Out-of-scope request → polite hand-off to the support line |
| Edge cases | `t9`, `t10`, `t11` | Prompt injection, ambiguous bug-or-refund, very short message |

## How the evaluation was run

1. `python generate-eval-dataset.py --tests-json harness-tests.json` invokes the
   harness once per test in a **fresh session** and writes
   [`output_eval_dataset.jsonl`](./output_eval_dataset.jsonl) in the format
   Bedrock Evaluations expects (`prompt`, `referenceResponse`, `modelResponses`).
2. The JSONL was uploaded to S3 (bucket from `cloudformation-testing.yaml`).
3. A Bedrock Evaluations job (LLM-as-a-judge, bring-your-own-inference) was run
   with the **correctness** metric.

## Results

**Correctness score:** _<!-- TODO: fill in the score from your Bedrock Evaluations results page, e.g. 0.9x -->_

**Observations:** _<!-- TODO: 3–5 sentences on what you saw — which routes scored well, any cases the judge marked down, and why. -->_

> Note to self (remove before submitting if you like): `t5`–`t7` were generated
> with placeholder `expected` text as their reference response, so the judge
> scored those three against non-final references. If your correctness score is
> pulled down, that's the likely cause — regenerate the JSONL after finalizing
> those references to get a cleaner score.
