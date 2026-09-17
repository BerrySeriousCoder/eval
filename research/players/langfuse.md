# Langfuse

Research checked: 2026-09-11. Documentation research; no deployment or SDK benchmark performed. The live documentation describes Langfuse v4.

## Product vision and users

Langfuse combines observability, evaluation, and prompt workflows. Engineers can score production activity, collect examples into datasets, and compare changes through experiments. It supports both human review and automated evaluation. [Evaluation overview](https://langfuse.com/docs/evaluation/overview)

The useful question for our project is how recorded execution becomes reusable evaluation data.

## Core abstraction and data model

| Object | Meaning |
| --- | --- |
| Observation | One step, such as generation, retrieval, or tool execution |
| Trace | Observations sharing a `trace_id`, representing one operation |
| Session | Related traces grouped into a conversation or workflow |
| Score | Evaluation or feedback associated with recorded behavior |
| Dataset item | Reusable input, optional expected output, and metadata |
| Experiment | Execution and evaluation against a dataset |

The current observability model stores trace-level attributes on observations, with SDK propagation. Its documentation describes a wide observations table rather than requiring a separately materialized trace entity for every query. Sessions group traces across interactions. [Observability data model](https://langfuse.com/docs/observability/data-model)

Do not confuse this documented v4 storage description with older diagrams. Conceptual objects and physical tables need not have a one-to-one relationship.

## Workflow and dashboard

Instrument an application, inspect its observations, add useful examples to a dataset, and run an experiment against changed code or prompts. Review automatic scores alongside manual annotations. Langfuse documents a CI action that fails a job when the experiment script raises a regression error. It is not necessary to treat Langfuse as production monitoring only. [Evaluation overview](https://langfuse.com/docs/evaluation/overview)

Dataset items have inputs, expected outputs, and metadata. Item changes create timestamped dataset versions; schema changes do not create those versions. Media attachments are supported for SDK experiments, while the reviewed page says UI experiments do not yet support media-bearing items. [Datasets](https://langfuse.com/docs/evaluation/experiments/datasets)

Our interpretation: record both dataset version and schema/configuration version if we need a fully reconstructable experiment.

## Evaluation methods

The current docs explicitly include code evaluators, LLM judges, scores submitted through APIs/SDKs, user feedback, and annotation queues. These feed score analytics and experiment comparison. A claim that Langfuse lacks deterministic checks or offline evaluation would be outdated. [Evaluation overview](https://langfuse.com/docs/evaluation/overview)

For our implementation, distinguish a human label from a model-generated label even when both use the same metric name. Agreement between them should be measured, not assumed.

## Architecture

The documented deployment contains:

- **Web:** UI and API handling.
- **Worker:** asynchronous event processing.
- **PostgreSQL:** transactional workloads.
- **ClickHouse:** analytical storage for observations and evaluation data.
- **Redis/Valkey:** queues and caching.
- **Object storage:** incoming events, media, and large exports.

Incoming trace batches are first persisted to object storage; queue references drive subsequent worker ingestion. This gives the ingestion pipeline a recovery path when downstream storage is unavailable. Langfuse distinguishes local Docker Compose use from production deployment arrangements. Some additional enterprise features require a license key. [Self-hosting](https://langfuse.com/self-hosting)

This architecture is documented, not inferred. Our design takeaway is the durable ingestion boundary, not a requirement to copy all of these services into a portfolio project.

## SDKs, APIs, and integrations

Observability builds on OpenTelemetry. Background export batches events; short-lived programs must flush before exiting to avoid losing buffered traces. [Observability data model](https://langfuse.com/docs/observability/data-model)

Python and JavaScript/TypeScript examples cover dataset creation and media handling. Dataset items can also come from captured observations. This provides an integration route for a custom agent and its evaluation runner. [Datasets](https://langfuse.com/docs/evaluation/experiments/datasets)

## Strengths, limitations, and cost

Our assessment:

- The observation/trace/session distinction is useful for diagnosing both a failed step and an unsuccessful conversation.
- Versioned examples connect debugging with repeatable testing.
- Self-hosting gives control over infrastructure but also creates operational work.
- Asynchronous export needs explicit shutdown handling; successful application execution does not itself prove telemetry was delivered.

The reviewed documentation establishes self-hosting and licensed add-ons. Exact cloud allowances, pricing, and enterprise entitlements were not audited. No claims about being cheaper or faster than competitors are made here.

## Voice relevance

Sessions and media-bearing dataset items can represent conversational evidence. Uploading an audio file does not itself compute transcription accuracy or interruption quality; those need an evaluator with the right inputs. This pass did not validate a complete built-in voice scoring workflow.

## Lessons and hand-coding exercise

Represent a call as a session, with individual operations containing nested spans. Implement a buffered exporter with `flush()` and a local ingestion endpoint that deduplicates event IDs. Start with one database.

Acceptance example: send the same event twice and terminate the producer after flushing. The viewer should show one event with the correct parent relationship. Later, convert one failed interaction into a versioned dataset case. This teaches trees, batching, idempotency, lifecycle handling, and schema design.
