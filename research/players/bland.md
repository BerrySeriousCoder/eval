# Bland AI

Research checked: 2026-09-11. Official documentation research; no calls, prompt publication, or paid experiments executed.

## Product vision and users

Bland's relevant evaluation surfaces include **Standards** for node behavior, **Testbed** for iterating from conversation evidence, and **Evals** for grading call quality. Developers can investigate a specific decision, preserve it as regression coverage, and separately evaluate entire calls. [Standards](https://docs.bland.ai/tutorials/standards), [Testbed](https://docs.bland.ai/tutorials/testbed), [Evals](https://docs.bland.ai/tutorials/evals)

For our project, the most useful lesson is the difference between checking a conversation component and checking the whole outcome.

## Core abstraction and data model

| Object | Meaning |
| --- | --- |
| Pathway | Graph of conversational nodes and edges |
| Node | Conversation phase with associated prompting/conditions |
| Standard | Scenario and expected behavior for a node component |
| Source conversation | Existing evidence used to construct a test |
| Eval agent | Judge for one call-quality dimension |
| Experiment | Batch of calls assessed by eval agents |

Pathway nodes can have dialogue, loop-condition, and extraction prompts. Standards target these separately. The documentation describes node processing in terms of a loop check, variable extraction, and dialogue generation. [Standards](https://docs.bland.ai/tutorials/standards)

This is a public conceptual model rather than a reconstruction of proprietary storage.

## Workflow: Testbed and Standards

Open a real or test conversation, select a node interaction, edit the relevant prompt or condition, and try the behavior against its preceding context. Testbed supports dialogue, loop-condition, and variable-extraction tests. It also allows editing conversation history to explore variations. [Testbed](https://docs.bland.ai/tutorials/testbed)

Standards execute ten scenarios/trials and compare the successful count with a threshold. Dialogue tests use simulated conversation; extraction and loop tests use the source transcript plus nine wording permutations retained for subsequent runs. Checks include a success-definition judge, expected loop behavior, and extraction checks using a prompt, regex, or exact value. [Standards](https://docs.bland.ai/tutorials/standards)

Testbed requires existing node standards to pass before publication. Publishing creates a Standard and publishes the prompt changes, so publication is an application mutation, not merely saving a test report. [Testbed](https://docs.bland.ai/tutorials/testbed)

Our interpretation: fixed fixtures make before/after comparisons easier, while adaptive dialogue explores additional routes. Neither establishes reliability across all possible conversations. A 9/10 trial result is an observation on those trials, not proof of a 90% production success rate.

## Workflow: call-level Evals

Bland Evals uses configurable LLM judges to assess batches of real or test calls. An eval agent has instructions and a **text** or **audio** modality: transcript judging and recording judging are distinct. It can return pass/fail or graded levels, and experiments provide per-call and aggregate results. Workbench setups save a reusable composition of evaluators and scoring settings. [Evals](https://docs.bland.ai/tutorials/evals)

This is direct evidence of audio evaluation support. It would be inaccurate to characterize Bland as only a call provider without evaluation tools.

Our assessment: judge configuration and score aggregation need their own versions. If a judge's rubric changes between runs, a change in average score need not mean the agent changed.

## Architecture, APIs, and integrations

The documented workflow can be summarized as:

```text
Pathway execution -> call evidence -> node Testbed -> Standards
                                 -> call-level Evals -> experiment results
```

This is a product workflow, not an assertion about database tables or backend services.

The Evals guide provides dashboard and API entry points. Separately, post-call webhooks deliver conversation data to an external server, including transcripts, extracted variables, and call duration. This is a possible ingestion boundary for our own evaluator. [Evals](https://docs.bland.ai/tutorials/evals), [Post-call webhooks](https://docs.bland.ai/tutorials/post-call-webhooks)

An initial integration should retain the raw webhook alongside a normalized call record. We would inspect the relevant detailed API contract before implementing retries, artifact retrieval, or event-specific assumptions.

## Strengths, limitations, and cost

Our assessment:

- Separating dialogue, extraction, and transitions supports targeted diagnosis.
- Reusing conversation context makes tests more realistic than isolated prompt examples.
- Fixed transcript variants prevent the test dataset from changing every time a node is tested.
- Passing a node test does not establish successful handoff between nodes or completion of an external action.
- An audio judge's verdict still needs validation against human review and measurable evidence.

Exact calling, simulation, and evaluation prices were not audited. Repetitions and model judging are usage drivers to account for in a future implementation. No claims about independent accuracy, lowest latency, or community complaints are included.

## Voice relevance

Bland explicitly distinguishes transcript and recording evaluation. Our project should preserve this distinction and add measured timing or external-state checks where needed. A judge assessing a recording and a deterministic check of response delay are different evaluators, even if both concern call quality.

## Lessons and hand-coding exercise

Build a small conversation state machine with independent transition and extraction checks. Save a failure's preceding context as a fixture, add reviewed paraphrases, and rerun the same fixtures after a code/prompt change.

Acceptance example: the caller gives an invalid date. The agent should stay in the collection state and request clarification. The report must show whether the failure came from extracting the date, accepting it, or transitioning early. This teaches graphs, state machines, fixture design, and component-level debugging.
