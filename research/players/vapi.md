# Vapi

Research checked: 2026-09-11. Official documentation research; no calls, simulations, or billable API jobs executed.

## Product vision and users

Vapi is a voice-agent platform whose testing tools operate against its **assistants** and **squads**. Its Evals test decisions at chosen conversational checkpoints; Simulations test complete conversations with an adaptive AI caller. These are different testing surfaces within the platform. [Evals quickstart](https://docs.vapi.ai/observability/evals-quickstart), [Simulations quickstart](https://docs.vapi.ai/observability/simulations-quickstart)

For our project, Vapi is both an integration candidate and an example of evaluation embedded in the agent-development workflow.

## Core abstraction and data model

| Object | Meaning |
| --- | --- |
| Assistant/squad | Target configuration under test |
| Eval | Mock conversation with checkpoints and validation |
| Eval run | Execution against the chosen target |
| Scenario/personality | Goal and behavior of a simulated caller |
| Simulation suite | Collection of simulations run against targets |
| Structured output | Schema-defined extraction or assessment from conversation evidence |

Evals carry role-labeled messages and use a `judgePlan` at selected assistant checkpoints. The quickstart documents saved targets and transient target configurations. [Evals quickstart](https://docs.vapi.ai/observability/evals-quickstart)

Simulation suites combine scenarios, personalities, success criteria, and repetitions. This conceptual map is not an internal storage schema. [Simulations quickstart](https://docs.vapi.ai/observability/simulations-quickstart)

## Workflow: decision tests

Provide the conversation state, ask the target to make its next decision, and validate the response or requested tool call. Judges include exact matching, regex, and AI-based judgment; tool checks can inspect function arguments. [Evals quickstart](https://docs.vapi.ai/observability/evals-quickstart)

These tests run at the text/model layer. They do not test the transcriber, voice, audio quality, or turn-taking. The preceding conversation fixes the situation being tested, making this useful for checks such as asking for missing information or refusing an invalid action. [Decision-test guidance](https://docs.vapi.ai/test/evals-best-practices)

Illustrative test for our project: the caller provides a date and time but no timezone. Pass if the agent asks for the timezone; fail if it proceeds to book using an assumption. This is a behavior test, not a requirement for one exact sentence.

## Workflow: full-conversation simulations

An AI caller follows a scenario and personality while interacting with the target. A suite can run in **chat** mode without speech processing or **voice** mode exercising speech, transcription, and turn-taking. Results expose success criteria and conversation evidence. [Simulations quickstart](https://docs.vapi.ai/observability/simulations-quickstart)

Simulations allow variable values, tool mocks, lifecycle webhooks, and reusable structured-output criteria. Unmocked tool execution must be considered when designing a test. [Advanced simulations](https://docs.vapi.ai/observability/simulations-advanced)

Our interpretation: simulated conversation diversity is valuable, but the simulator itself is another model with blind spots. It should supplement curated scenarios and real-call review.

## Evaluation methods and evidence

Structured outputs use a schema and model instructions to derive fields from conversation context. The documented template variables include transcripts, messages, timestamps, call duration, and the assistant system prompt. [Structured outputs](https://docs.vapi.ai/assistants/structured-outputs)

A Boolean field saying “appointment confirmed” is not proof that a calendar record exists. Vapi's simulation quickstart explicitly distinguishes a conversational confirmation check from checking external state, and notes that its example criterion does not grade pronunciation or interruptions. [Simulations quickstart](https://docs.vapi.ai/observability/simulations-quickstart)

For our implementation, keep three checks separate: intended tool arguments, handling the tool's response, and the resulting record in a sandbox business system.

## Architecture, SDK/API, and dashboard

The public testing workflow is:

```text
Mock state OR adaptive caller -> assistant/squad
    -> conversation/tool evidence -> judges/structured outputs -> results
```

The Evals documentation provides dashboard and API workflows. It distinguishes individual saved Evals from Simulation suites; grouping Evals for CI is managed by the user's automation rather than a native Eval-suite feature. [Evals quickstart](https://docs.vapi.ai/observability/evals-quickstart)

Structured-output CRUD has an API surface, and simulation lifecycle hooks can notify an external system. Those are potential adapter boundaries for our project. This research does not establish Vapi's private storage or scheduling implementation. [Structured outputs](https://docs.vapi.ai/assistants/structured-outputs), [Advanced simulations](https://docs.vapi.ai/observability/simulations-advanced)

## Strengths, limitations, and cost

Our assessment:

- Checkpoint tests isolate a decision; adaptive simulations exercise a journey.
- Tool mocks make specific success and failure conditions repeatable.
- A passing transcript criterion does not establish acoustic quality.
- Native tests target Vapi configurations; a provider-neutral project needs its own adapter interface.

Voice simulations exercise more components than chat simulations. Exact prices, plan limits, and provider charges were not audited; do not budget from old per-minute claims.

## Lessons and hand-coding exercise

Build two runner modes: replay a fixed conversation to a checkpoint, and run a bounded conversation with a simulated user. Give the latter a maximum turn count and preserve the simulator configuration.

Acceptance example: inject a booking-tool error. The agent must explain the failure and avoid claiming success. Report the tool evidence separately from the judge's verdict. Later add audio artifacts without treating their presence as an automatic audio pass. This teaches state machines, mocks, orchestration, and evidence-based assertions.
