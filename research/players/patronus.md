# Patronus AI

Research checked: 2026-09-11. Documentation research only; no API execution or independent judge benchmark.

Evidence limitation: several conceptual/evaluator pages returned HTTP 401 to the browser. The experiment, dataset, weighting, and export guides were readable. Where the evaluator catalog is discussed below, it is based on search-indexed official documentation and is labeled accordingly.

## Product vision and users

Patronus provides an evaluation workflow in which developers run task outputs through evaluators and inspect experiment results. The experiment runner takes a dataset, evaluators, and an optional task function, with controls such as concurrency and project/experiment names. [Python experiment guide](https://docs.patronus.ai/docs/experiments/run_python)

For our project, the important idea is to make the evaluator an independent, replaceable component. The model answering a user and the system judging that answer are separate roles.

## Core abstraction and data model

| Object | Meaning |
| --- | --- |
| Dataset row | Task evidence and optional expected answer |
| Task | Optional function producing an output |
| Evaluator | Rule or service assessing the evidence |
| Experiment | Organized execution over rows and evaluators |
| Evaluation result | Per-row assessment retained for inspection/export |

The upload guide maps data into `task_input`, `task_output`, `task_context`, and `gold_answer`. It supports CSV/JSONL uploads and downloading hosted datasets. Existing outputs can therefore be represented alongside expected answers. [Dataset upload](https://docs.patronus.ai/docs/datasets/upload)

The field map is documented; it is not a statement about Patronus's internal database tables.

## Workflow and dashboard

Prepare the dataset, optionally run a task for each row, pass its output to evaluators, and collect results. The Python runner supports synchronous and asynchronous use. [Python experiment guide](https://docs.patronus.ai/docs/experiments/run_python)

The results guide documents per-row scores in the UI, aggregate statistics, and DataFrame exports to formats including CSV and JSONL. This lets a developer continue analysis outside the platform. [Statistics and export](https://docs.patronus.ai/docs/experiments/experiment-stats)

Our interpretation: retain row identity and raw results before computing summaries. A pass rate is only interpretable when the denominator, excluded errors, and evaluated examples are known.

## Evaluation methods

The readable weighting guide demonstrates three evaluator forms: remote evaluators, function adapters, and structured evaluator classes. It also makes a narrower point than a generic “weighted scoring” claim: evaluator weights are supported within experiments and stored as experiment metadata; the page does not establish that arbitrary standalone calls produce a combined weighted grade. [Evaluator weights](https://docs.patronus.ai/docs/experiments/evaluator_weights)

Search-indexed official catalog content describes `RemoteEvaluator`, a Lynx hallucination evaluator, configurable judge criteria, and explanation strategies such as always, never, or on failure. Direct catalog retrieval failed, so exact current evaluator availability and identifiers should be checked before integration. [Official evaluator catalog, indexed content](https://docs.patronus.ai/docs/evaluators/patronus)

This pass did not reproduce vendor claims of superior judge accuracy. A specialized judge still needs validation on our domain and human-labeled examples.

## Architecture, SDKs, and integrations

The documented experiment interface implies the following application-facing flow:

```text
Local/loaded examples -> optional task -> evaluator calls -> results
                                             |
                                  remote evaluation service
```

The diagram is a conceptual client workflow, not a private infrastructure diagram. The Python guide confirms SDK-based execution; the documentation also links a TypeScript experiment path, but Python/TypeScript feature parity was not audited. [Python experiment guide](https://docs.patronus.ai/docs/experiments/run_python)

The evaluator adapters support integrating our own code checks into the same experiment as remote judgments. [Evaluator weights](https://docs.patronus.ai/docs/experiments/evaluator_weights)

## Strengths, limitations, and cost

Our assessment:

- Replaceable evaluator implementations make a useful boundary for testing judges independently.
- Result export supports analysis and debugging without depending entirely on a dashboard.
- A remote evaluator adds failure modes such as timeout, rate limiting, or unavailable configuration.
- Weighting requires an explicit aggregation policy; critical failures should not disappear inside an average.

Exact pricing, hosted-service entitlements, self-hosting availability, and the current licensing of separate SDK/model artifacts were not established. The existence of a local Python runner is not evidence that proprietary remote judges execute locally for free.

## Voice relevance

Transcript and tool evidence could be mapped into task fields for our own evaluator integration. This pass did not verify a complete audio, timing, or telephony simulation workflow. Do not label the platform text-only based on that research limit.

## Lessons and hand-coding exercise

Implement one evaluator interface with two adapters: a local rule and a remote judge. Normalize their results into a structure containing score or label, verdict, explanation, evaluator version, and execution status.

Acceptance example: compare judge labels against a small human-labeled set, displaying false positives and false negatives. Inject a remote timeout and ensure it becomes an evaluator error rather than an agent failure. This teaches adapters, error handling, confusion matrices, and evaluator calibration.
