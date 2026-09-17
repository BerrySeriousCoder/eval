# Player research reading guide

Research pass: 2026-09-11.

The purpose of these profiles is to help you understand evaluation systems and build your own implementation. Each new profile covers product purpose, core objects, workflow, evaluation methods, architecture/integration evidence, limitations, voice relevance, and a hand-coding exercise.

## Suggested reading order

This order follows the concepts needed for our project, rather than ranking the products.

| Read | Profile | Question to take into the implementation |
| --- | --- | --- |
| 1 | [Braintrust](braintrust.md) | How should data, task execution, and scoring fit together? |
| 2 | [OpenAI Evals](openai-evals.md) | How do a test definition, run, and result differ? |
| 3 | [Arize Phoenix](arize-phoenix.md) | How can execution traces explain failed evaluations? |
| 4 | [Langfuse](langfuse.md) | How should ingestion, conversations, and dataset versions work? |
| 5 | [Galileo](galileo.md) | What scope and evidence does each metric require? |
| 6 | [Patronus AI](patronus.md) | How do we replace and validate an evaluator? |
| 7 | [Vapi](vapi.md) | How do checkpoint tests differ from simulated journeys? |
| 8 | [Retell AI](retell.md) | How do we ingest call evidence and interpret voice metrics? |
| 9 | [Bland AI](bland.md) | How can we isolate extraction and conversation-transition failures? |

[LangSmith](langsmith.md) and [DeepEval](deepeval.md) remain earlier drafts that need source verification. They are not included in this completed documentation pass.

## How to read the evidence

- Inline links identify the official source for a product claim. The date above is the research date, not the source's publication date.
- “Our assessment,” “our interpretation,” and exercises are design analysis, not vendor functionality or measured benchmarks.
- Workflow diagrams show public behavior or explicitly proposed adapters. Private infrastructure is not guessed.
- “Not verified” means the research did not establish a capability. It does not mean the capability is absent.
- Documentation proves that a feature is documented. It does not prove accuracy, latency, ease of use, or availability in every account.
- Exact prices are omitted unless directly checked and scoped. In particular, a QA add-on price is not a total calling price.
- No SDK examples were executed, paid tests run, or authenticated dashboards benchmarked. Patronus's inaccessible pages are identified in its profile.

## Decisions to carry into the next research stage

These are proposed principles for our implementation, not a finalized specification:

1. Keep test inputs, generated outputs, and evaluator results as separate records.
2. Version the candidate, dataset, rubric, and threshold so a comparison has a clear meaning.
3. Distinguish a behavioral failure from a runner error, a judge error, and missing evidence.
4. Show individual regressions before relying on aggregate scores.
5. Keep conversational claims separate from tool results and verified business state.
6. Give text, audio, timing, and outcome checks explicit input requirements.
7. Treat a simulator and a judge as systems that need validation themselves.

Each profile ends with a bounded exercise. Those exercises are options for the build roadmap, not nine separate products to implement.

## Remaining research boundaries

The remaining-player documentation pass is complete within the stated evidence limits. Before implementation, synthesize the profiles into the problem statement, product thesis, comparison documents, and a small first milestone. Revalidate the two earlier player drafts before using them in that comparison. A documented feature gap or an integration inconvenience alone does not establish market demand.
