# Galileo

Research checked: 2026-09-11. Documentation research only; no SDK runs or independent metric-accuracy tests.

## Product vision and users

Galileo supports evaluation of prompts, models, and application code through datasets, experiments, and metrics. It connects evaluation results to execution traces so engineers can inspect why a configuration performed differently. [Experiments basics](https://docs.galileo.ai/sdk-api/experiments/experiments)

Its useful lesson for this project is that a metric needs a scope and required evidence, not just a name such as “quality.”

## Core abstraction and data model

| Object | Meaning |
| --- | --- |
| Project | Container for experiments |
| Dataset | Inputs and optional ground-truth values |
| Prompt template/custom function | Candidate behavior to execute |
| Experiment | Evaluation of a configuration against examples |
| Experiment log stream | Recorded traces produced by an experiment |
| Trace/span | Execution evidence at different levels |
| Metric | Assessment applicable to particular node types |

The documented experiment model has one log stream per experiment and a trace per dataset row. Results include inputs, outputs, system metrics such as latency/token usage, and evaluation metrics. [Experiments basics](https://docs.galileo.ai/sdk-api/experiments/experiments)

This is a conceptual data model. The sources do not expose a complete proprietary storage schema.

## Workflow and dashboard

Define examples and select metrics, then evaluate a prompt template or custom application function. Inspect aggregate results and drill down to traces. The experiment workflow also accommodates already-generated output, allowing scoring without making the original model call again. [Experiments basics](https://docs.galileo.ai/sdk-api/experiments/experiments), [Run experiments in code](https://docs.galileo.ai/sdk-api/experiments/running-experiments)

Our interpretation: separate generating outputs from evaluating outputs. Otherwise, changing the judge also changes the evidence being judged, making comparisons harder to explain.

## Evaluation methods

Galileo documents built-in metrics, custom code, and custom LLM judges. Metrics apply to different scopes, including sessions, traces, and particular span types. Its metric framework covers agent behavior, response quality, RAG, multimodal quality, and other categories. LLM-based metrics require a configured model integration or the relevant Luna model setup. [Metrics overview](https://docs.galileo.ai/concepts/metrics/overview)

The docs describe **Autotune** as using feedback to improve alignment of metrics with domain expectations. That is a documented calibration workflow, not independent evidence that the resulting judge is accurate on our data. [Metrics overview](https://docs.galileo.ai/concepts/metrics/overview)

For an implementation, every metric should declare its required inputs. A retrieved-context check should not silently run on an example with no retrieved context and produce a plausible-looking score.

## Voice and multimodal relevance

Galileo explicitly documents multimodal quality evaluation, including detecting conversational overlap and barge-in. It directs users to log the relevant media before applying those metrics. Therefore, describing Galileo as text-only or claiming that no existing evaluator addresses turn-taking would be wrong. [Multimodal quality metrics](https://docs.galileo.ai/concepts/metrics/multimodal-quality/multimodal-quality-overview)

Our assessment: a built-in metric is a candidate to evaluate, not a benchmark result. We still need to establish which audio format it expects, how speakers and timing are represented, and how its labels agree with human review. This pass did not validate those details or independently reproduce its accuracy.

## Architecture, SDKs, and integrations

The documented client flow is:

```text
Dataset -> prompt generation / custom application / saved output
        -> traced experiment records -> metric calculation -> inspection
```

Python uses `run_experiment`; TypeScript uses `runExperiment`. The runner creates session/trace context for each dataset row. The docs warn that manually instrumented application code must respect that existing context rather than independently starting or ending the same experiment trace. [Run experiments in code](https://docs.galileo.ai/sdk-api/experiments/running-experiments)

This is a useful integration constraint: an evaluation wrapper should compose with the application's instrumentation. The reviewed sources do not establish Galileo's private database, queue implementation, or service topology.

## Strengths, limitations, and cost

Our assessment:

- Metrics with explicit execution scope are more informative than one undifferentiated conversation score.
- Scoring existing output makes evaluator iteration easier to isolate.
- Feedback-driven judge calibration deserves its own dataset and history.
- Broad metric availability can distract from selecting a few checks tied to actual product failures.

Current plan prices, Luna availability by plan, and self-deployment entitlements were not audited. Model-based evaluation adds inference usage; no vendor latency or accuracy superiority claims were reproduced.

## Lessons and hand-coding exercise

Create a metric registry with `name`, `version`, `scope`, `required_inputs`, and `evaluate`. Keep output generation optional so an experiment can evaluate saved results.

Acceptance example: run a tool-argument check on a tool span and a conversation-outcome check on the full session. A missing recording must yield “not evaluated” for an audio check, with an explanation. It must not become a zero or a pass. This teaches interfaces, input validation, context propagation, and metric semantics.
