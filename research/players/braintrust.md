# Braintrust

Research checked: 2026-09-11. Documentation research; no paid account, SDK execution, or performance benchmark used.

## Product vision and users

Braintrust connects application tracing, evaluation, and iteration. Its evaluation workflow moves from playground experiments to saved runs, CI checks, production scoring, and new datasets built from interesting production examples. The intended user is an engineer or team deciding whether a prompt, model, or application change improves behavior. [Evaluation overview](https://www.braintrust.dev/docs/evaluate)

For our project, study how a test runner becomes a usable experiment system: running checks is only the beginning; people also need to inspect and compare results.

## Core abstraction and data model

The central evaluation contract has three parts: **data**, **task**, and **scorers**. Data supplies inputs and optional expected answers; the task is the application function; scorers judge its output. A task can be an entire agent rather than one model call. [Evaluation overview](https://www.braintrust.dev/docs/evaluate)

| Object | Meaning |
| --- | --- |
| Dataset case | Reusable input, optional expected result, and metadata |
| Task | Code that produces the actual result |
| Experiment | Saved record of an evaluation run |
| Scorer | Numeric quality assessment |
| Classifier | Categorical label, such as intent, without an implied quality ranking |
| Trace/span | Execution context available for inspection and evaluation |

Scorers receive fields including `input`, `output`, `expected`, `metadata`, and `trace`. Numeric scores use a 0–1 range; classifiers return categories. A classification such as “billing” is therefore different from a correctness score. [Scorers and classifiers](https://www.braintrust.dev/docs/evaluate/write-scorers)

This is a conceptual map of public objects, not a reconstruction of proprietary SQL tables.

## Workflow and dashboard

1. Prepare examples representing the behavior to preserve.
2. Choose the task implementation and scoring rules.
3. Run an experiment in code, the UI, or CI.
4. Inspect individual outputs and their traces.
5. Compare the experiment against a previous run.

Braintrust describes experiments as immutable snapshots, while playground results can be overwritten during iteration. The distinction makes an experiment a shareable record of what was tested. [Experiments](https://www.braintrust.dev/docs/evaluate/run-evaluations)

Our interpretation: comparing runs requires stable case identity and configuration records. A dashboard that shows only average scores would hide which examples changed.

## Evaluation methods

Braintrust supports built-in Autoevals, custom code, and LLM judges. Scoring can operate on an individual span or an entire trace; production scoring also has a group scope for related traces. The documented scorer workflow includes checking graders against examples and versioning them. [Scorers and classifiers](https://www.braintrust.dev/docs/evaluate/write-scorers)

An illustrative booking evaluation could use a code check for timezone arguments and a separate judge for whether the agent clearly explained a tool failure. These are our proposed checks, not named Braintrust metrics.

Do not read the vendor's description of reproducible experiments as a guarantee of identical model outputs. A saved result is reproducible evidence; rerunning a stochastic task can produce a different result.

## Architecture, SDKs, and integrations

The documented deployment separates a **data plane** from a **control plane**. The data plane holds evaluation and trace content and includes an API, PostgreSQL, Redis, object storage, and Brainstore. The control plane handles the UI, identity, and organizational metadata. In the documented self-hosted arrangement, the customer operates the data plane and Braintrust continues to host the control plane. SDK traffic goes directly to the data plane. [Self-hosting architecture](https://www.braintrust.dev/docs/admin/self-hosting)

Python and TypeScript are among the SDK paths documented for defining and sharing scorers; the docs also list other language SDKs. Code-based experiments and CI integration let teams evaluate their own application functions without expressing everything as a hosted prompt. [Scorers and classifiers](https://www.braintrust.dev/docs/evaluate/write-scorers), [Experiments](https://www.braintrust.dev/docs/evaluate/run-evaluations)

## Strengths, limitations, and cost

Our assessment:

- Separating tasks from scoring gives a clean boundary for custom application code.
- Saved experiments and trace inspection make a useful debugging workflow.
- Operating the documented self-hosted data plane is more infrastructure than our first learning milestone needs.
- LLM judging introduces another model execution whose failures and cost should be measured separately.

Self-hosting here does not mean a fully independent local installation of every platform component. Current plan entitlements and exact prices were not verified in this pass. No unsupported community complaints or comparative performance claims are included.

## Voice relevance

Conversation or tool traces can supply evidence to our own scoring logic. This research did not establish a built-in telephony simulator or a specific acoustic metric suite. That is an unverified capability, not evidence that the product cannot support audio.

## Lessons and hand-coding exercise

Build a small `evaluate(dataset, task, scorers)` interface. Store outputs independently of scores so you can rescore saved outputs without rerunning the agent. Add a baseline comparison keyed by case ID, showing improvements, regressions, missing results, and execution errors.

Acceptance example: an experiment improves on four cases but regresses on one critical booking case. The report must expose that case even if its mean score rises. This exercises interfaces, async execution, joins, and result aggregation.
