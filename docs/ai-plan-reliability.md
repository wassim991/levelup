[← Case study](../README.md) · [Run the evidence](verification.md)

# AI plans: budget approval is not output validity

**A full plan needs enough room to satisfy its contract.** Shortening a response is only a useful fallback when the feature can still work with the shorter result.

## The Level Up incident

I traced failed plans to shared backend preflight logic that could cap structured generation at 120 tokens. This was application budget logic running through the Supabase backend, not a Supabase platform limit on model output.

The budget fallback allowed a request to proceed with too little room for a complete daily or weekly plan. The failure was upstream of JSON validation: retrying the same generation under the same cap would not fix that constraint.

## The change and its tradeoff

Full-plan requests keep their required output budget when it fits. Otherwise, preflight rejects the request before model generation. Short-form features can retain a different degradation policy.

That choice can mean showing a budget failure instead of producing a partial response. For a plan the app must execute, an explicit failure is preferable to treating an incomplete structure as usable.

I kept output validation as a separate boundary. Daily-plan generation gets client/server schema checks and one compact repair attempt; exhausted recovery becomes a typed failure. Budget approval alone cannot guarantee valid model output.

A follow-up correction restored weekly allowance handling in the shared preflight function. Changing shared policy meant checking the daily and weekly callers, not just the request that first exposed the bug. Live preflight checks covered those paths; they were not a broad model-quality evaluation.

## What you can inspect publicly

The [TypeScript pipeline][pipeline] isolates **output validation and bounded recovery**. It does not reproduce the private quota function or the Level Up plan schema.

1. Validate options and input size before dispatch.
2. Ask an injected provider for JSON, then parse and validate it against a [strict schema][schema].
3. Permit one repair for invalid JSON or schema violations.
4. Stop immediately for provider failure, oversize output, cancellation, or deadline expiry.
5. Return a discriminated result with a typed value or failure code and the actual call count.

Both calls share one deadline. Repair receives bounded prior output and filtered schema diagnostics; the caller-facing failure does not expose raw provider exceptions or invalid output. The provider adapter remains a trusted boundary because it receives that content.

## Tests worth opening

In [the regression suite][pipeline-tests]:

| Test | What it establishes |
| :--- | :--- |
| `failed repair returns final reason and stops at two calls` | Exhausted recovery cannot silently become an unbounded retry loop. |
| `repair shares the original deadline rather than receiving a fresh budget` | Repair consumes the same total time allowance. |
| `in-flight cancellation settles even if provider ignores abort` | The caller can stop waiting for an uncooperative provider. |
| `abort before provider microtask reports zero actual attempts` | Attempt counts reflect calls dispatched, rather than loop iterations. |

## Limits

Tests use scripted provider responses. They establish control flow, not plan usefulness, factual accuracy, or real-provider reliability. Passing a schema is not a model-quality score. The demo's size limits count UTF-16 code units, not tokens; network adapters must enforce byte limits while reading. Cancellation settles the caller but cannot forcibly terminate arbitrary provider work.

**The lesson:** preserve the feature's contract before optimizing its cost, then test recovery separately from admission.

[pipeline]: https://github.com/wassim991/llm-structured-output-pipeline/blob/3eccf725b333367cd9539aa980ade9d8d294c7b9/src/pipeline.ts
[pipeline-tests]: https://github.com/wassim991/llm-structured-output-pipeline/blob/3eccf725b333367cd9539aa980ade9d8d294c7b9/tests/pipeline_test.ts
[schema]: https://github.com/wassim991/llm-structured-output-pipeline/blob/3eccf725b333367cd9539aa980ade9d8d294c7b9/src/schema.ts
