# OpenAI Evals

Research checked: 2026-09-11. Documentation research only; no evaluation jobs executed.

## Current status and scope

**The hosted Evals platform is being deprecated.** OpenAI's deprecations page specifies read-only access for existing evals on **October 31, 2026**, followed by scheduled shutdown of the Evals dashboard and API on **November 30, 2026**. It also places graders documented for eval workflows within this transition. This makes the hosted service a design reference rather than a suitable new dependency for our project. [Deprecation timeline](https://developers.openai.com/api/docs/deprecations)

Two different systems have been called OpenAI Evals:

| System | Interface and purpose |
| --- | --- |
| Earlier open-source framework | Python evaluation implementations, YAML registry entries, JSONL data, and a CLI |
| Hosted platform/API | Eval definitions, data-source configuration, graders, and managed runs |

The older framework is documented in an official Cookbook walkthrough. The hosted platform is documented separately. The hosted shutdown notice should not be presented as a verified statement about the repository's maintenance status. [Framework walkthrough](https://developers.openai.com/cookbook/examples/evaluation/getting_started_with_openai_evals), [Hosted evals guide](https://developers.openai.com/api/docs/guides/evals)

## Product vision and users

The common aim is to express expected model behavior as tests and evaluate changes against examples. Engineers can assess task-specific behavior instead of relying solely on a model's general benchmark results. The hosted workflow separates describing the task, executing tests, and analyzing results. [Hosted evals guide](https://developers.openai.com/api/docs/guides/evals)

## Core abstraction and data model

In the hosted API, an **eval definition** combines a data schema and testing criteria. A **run** applies that definition to concrete data and a generation configuration. Graders reference dataset fields through `item` and generated fields through `sample`. [Create eval reference](https://developers.openai.com/api/reference/java/resources/evals/methods/create)

Conceptually:

```text
Eval definition: input schema + grading criteria
    -> Run: dataset + candidate configuration
        -> Per-example output + grading result
            -> Summary and failure inspection
```

This diagram describes the public workflow; it makes no claim about OpenAI's internal database or queue design.

## Earlier framework workflow

The Cookbook describes registering an eval through YAML, connecting it to JSONL examples and an implementation class, and executing it with `oaieval` or `oaievalset`. Structured logs contain run specifications, sample events, and a final report. Its SQL example also explains that a model's judgment of SQL correctness is different from actually executing the SQL. [Framework walkthrough](https://developers.openai.com/cookbook/examples/evaluation/getting_started_with_openai_evals)

These are historical interface descriptions, not newly tested installation instructions. The examples' old model identifiers should not be copied into a current project without checking availability.

## Hosted workflow and integration

Create the evaluation definition, provide data, create a run, then retrieve its status and results. The guide supports API-based operation and a dashboard, and documents completion/failure/cancellation webhooks. Python and JavaScript examples demonstrate the application-facing workflow. [Hosted evals guide](https://developers.openai.com/api/docs/guides/evals)

External-model evaluation includes custom HTTPS endpoints compatible with Chat Completions, subject to organization configuration. The reviewed external-model guide explicitly says tool calls are not supported in that workflow. Do not generalize this into “OpenAI Evals only supports OpenAI models.” [External models](https://developers.openai.com/api/docs/guides/external-models)

## Evaluation methods

The grader documentation describes string comparisons, text similarity, model-based scoring, and Python code execution. These represent different kinds of evidence: exact output equality, approximate overlap, a judge's assessment, or an executable rule. [Graders](https://developers.openai.com/api/docs/guides/graders)

The API also documents label-model graders: the allowed labels and passing labels are separate configuration. This is useful when a result is naturally categorical rather than a continuous score. [Create eval reference](https://developers.openai.com/api/reference/java/resources/evals/methods/create)

OpenAI's evaluation guidance discusses judge position and verbosity biases and recommends checking evaluation behavior against human judgment. A model's score is not ground truth simply because it is returned in structured JSON. [Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)

## Strengths, limitations, cost, and voice relevance

Our assessment: the schema/definition/run separation is a useful API design lesson. The hosted lifecycle is a material limitation for new adoption. Remote target generation and remote judging are separate usage drivers; current pricing was not audited.

The reviewed sources do not establish a full telephone-call simulator with transport, interruption, and timing evaluation. Text or model grading can assess conversational content, but acoustic and business-outcome evidence needs its own collection and scoring path.

## Lessons and hand-coding exercise

Implement a versioned evaluation definition with a schema, evaluator IDs, and thresholds. Keep run records separate and preserve every case result. Add states such as queued, running, completed, failed, and canceled, independently of whether the candidate passed.

Acceptance example: a judge timeout is an evaluation error, while a successfully graded wrong answer is a failed check. The API and report must distinguish them. This exercises schema validation, state machines, asynchronous jobs, and honest reporting.
