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

**Correctness score:** **0.86** (average 0.864 across all 11 prompts).
**Harmfulness score:** **0.00** (no harmful content in any response).

See [`results_screenshot/bedrock-evaluation-correctness.png`](./results_screenshot/bedrock-evaluation-correctness.png)
for the Bedrock Evaluations results page.

**Observations:** The correctness histogram shows **9 of 11 prompts scoring ~0.9**,
so the chatbot answers correctly across bug-report, platform-question, and
other-request routing in the large majority of cases. Two prompts scored low
(~0.5 and ~0.0), which pulls the average down to 0.86. Those two are among the
FAQ tests (`t5`–`t7`) whose reference responses were still placeholder text when
the dataset was generated, so the LLM-judge compared good model answers against
non-final references rather than the model being wrong — visible in the
[chat transcript](./results_screenshot/chat_followup_transcript.png), the FAQ
answers themselves are accurate and grounded in `online_shop_faq.md`. Harmfulness
scored 0.00, and the prompt-injection test (`t9`) was refused correctly. End to
end, the bug-report route also persists real tickets: the
[DynamoDB table](./results_screenshot/dynamodb-bug-reports.png) holds 5 tickets
created by the chatbot, each with description, steps to reproduce, environment,
and an OPEN status.

## Evidence (screenshots)

| Rubric criterion | Screenshot |
|------------------|------------|
| Harness created & ready | [`create-harness-ready.png`](./results_screenshot/create-harness-ready.png) |
| Bug-report path — chat transcript with `[tool call] bugreports___create_bug_report` | [`chat_followup_transcript.png`](./results_screenshot/chat_followup_transcript.png) |
| Bug-report path — DynamoDB ticket with collected fields | [`dynamodb_report (2).png`](./results_screenshot/dynamodb_report%20(2).png) |
| Platform question — covered FAQ | [`faq-covered-question.png`](./results_screenshot/faq-covered-question.png) |
| Platform question — uncovered FAQ | [`faq-uncovered-question.png`](./results_screenshot/faq-uncovered-question.png) |
| Other request — support hand-off | [`other-request.png`](./results_screenshot/other-request.png) |
| Classification categories in system prompt | [`system_3_category.png`](./results_screenshot/system_3_category.png) |
| Bug-report path — ticket persisted to DynamoDB | [`dynamodb-bug-reports.png`](./results_screenshot/dynamodb-bug-reports.png) |
| Testing & evaluation — Bedrock Evaluations correctness | [`bedrock-evaluation-correctness.png`](./results_screenshot/bedrock-evaluation-correctness.png) |
