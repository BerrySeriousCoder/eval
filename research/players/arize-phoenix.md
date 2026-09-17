# Arize Phoenix

Research checked: 2026-09-11. Documentation research; no local deployment or benchmark performed.

## Product vision and users

Phoenix helps engineers inspect AI execution, score behavior, and compare application changes. It combines tracing, evaluations, prompt iteration, datasets, and experiments. Arize also offers **Arize AX**, a separate managed enterprise platform; do not attribute every AX feature or its infrastructure to Phoenix. [Phoenix overview](https://arize.com/docs/phoenix/)

For our hand-coded project, Phoenix is particularly useful as a reference for connecting a trace viewer to an experiment runner.

## Core abstraction and data model

| Object | Meaning |
| --- | --- |
| Trace | An execution consisting of related spans |
| Span | A model call, retrieval, tool call, or other operation |
| Evaluation/annotation | A quality assessment attached to recorded behavior |
| Dataset example | Input used to test application behavior |
| Task | Function that executes the candidate application |
| Experiment | Task executions and evaluations across examples |

Phoenix receives OpenTelemetry traces and uses OpenInference instrumentation for AI-specific context. Its documentation includes framework and provider integrations and evaluations from model judges, code, or human labels. [Phoenix overview](https://arize.com/docs/phoenix/)

These are public abstractions. The table is not an exact schema dump.

## Workflow and dashboard

The TypeScript walkthrough begins with captured failures, places them in a dataset, defines an improved agent as the experiment task, and evaluates its outputs. The UI lets the developer inspect the resulting outputs and scores. The task and evaluator are independent functions, so the application can change while the checking criteria remain stable. [Experiment walkthrough](https://arize.com/docs/phoenix/get-started/ts-get-started-datasets-and-experiments)

Our interpretation: failed examples are valuable regression coverage, but a suite made only from failures is not representative of overall traffic. Preserve a broader held-out set before claiming a general quality improvement.

## Evaluation methods

Phoenix documents LLM-based evaluation, code checks, human annotation, and reusable dataset evaluators. Its overview also links integrations with external evaluation libraries. This means collecting traces does not require committing to one scoring implementation. [Phoenix overview](https://arize.com/docs/phoenix/)

The experiment example uses a classification evaluator with explicit labels mapped to numeric values. This makes aggregation straightforward, but the score still represents the chosen rubric. [Experiment walkthrough](https://arize.com/docs/phoenix/get-started/ts-get-started-datasets-and-experiments)

For our booking agent, an illustrative evaluator could check whether the expected tool was requested, while a separate evaluator checks whether the final message truthfully describes the tool result. A correct final sentence should not hide a failed underlying operation.

## Architecture and deployment

Phoenix's documented architecture is a containerized application with a web UI, trace collector, and SQL backend. SQLite is the default for local use; PostgreSQL is the documented production option. A Phoenix instance represents one tenant, with access governed by roles. The architecture page separately identifies AX's analytical database, so it should not appear in a Phoenix deployment diagram. [Architecture](https://arize.com/docs/phoenix/self-hosting/architecture)

```text
Instrumented application -> trace collector -> SQL storage -> UI
Experiment task + evaluators -> saved experiment results -> comparison
```

This is a simplified conceptual flow. It omits deployment details and does not imply that every evaluator runs in the collector process.

Docker and Helm deployment artifacts are documented, including version-pinned images. Self-hosting also has operational configuration for authentication, access controls, retention, and networking. [Self-hosting](https://arize.com/docs/phoenix/self-hosting)

## SDKs, APIs, and integrations

The TypeScript experiment example uses `@arizeai/phoenix-client` for datasets and experiment execution and `@arizeai/phoenix-evals` for evaluation. A custom application function supplies the output. [Experiment walkthrough](https://arize.com/docs/phoenix/get-started/ts-get-started-datasets-and-experiments)

OpenTelemetry ingestion is a useful interoperability boundary: a custom application can export spans without adopting a particular agent framework. Our own project can initially use a smaller internal format and add an adapter when needed.

## Strengths, limitations, and cost

Our assessment:

- A local SQL-backed deployment is an understandable starting point for learning system internals.
- Traces and experiments can share evidence without becoming the same object.
- Production operation still requires persistence, backups, and lifecycle management.
- Self-hosting a collector does not remove the inference cost of remote model judges.

Exact hosted pricing and the current repository license were not audited. “Built with the open-source community” is not a substitute for reading the applicable license before reusing source code.

## Voice relevance

Our adapter could associate transcription, model, tool, and speech-generation spans with a call. This research did not validate a ready-made telephony simulator or acoustic metric suite in Phoenix. Instrumentation supplies execution evidence; it does not automatically establish what the caller heard.

## Lessons and hand-coding exercise

Build a minimal trace collector and a tree viewer. Attach evaluation records by execution ID, then implement a dataset runner that reuses those records for comparison. Start with one SQL database.

Acceptance example: a retrieval step fails but the agent still produces a fluent answer. The viewer must reveal the failed child span, and the experiment must preserve both the answer and the failure evidence. This exercises tree traversal, relational modeling, instrumentation, and debugging interfaces.
