# AI Evaluation Research

> Learn AI evaluation well enough to hand-code, debug, and explain an evaluation project.

## Project purpose

This research supports a personally implemented evaluation system and renewed coding practice. The aim is to understand the engineering decisions, build the core implementation by hand, and develop material for technical interviews. Commercial differentiation is a secondary research question, not a prerequisite for a useful project.

## Player research status — 2026-09-11

The nine remaining player profiles now have documentation-based research, source links, evidence limitations, and implementation exercises. Start with the [player reading guide](players/README.md).

LangSmith and DeepEval are earlier drafts. Their claims have **not** been reverified in this pass; do not treat their pricing, feature exclusions, or unsourced community complaints as current facts.

These profiles are documentation studies, not hands-on product benchmarks. Product-thesis, cross-platform comparison, and architecture decisions remain a separate stage of research.

## Why this repository exists

Large Language Models are evolving at an unprecedented pace.

Every few weeks a new model is released.

Engineering teams constantly face questions like:

- Should we switch to the latest model?
- Did our new prompt improve quality?
- Is latency better or worse?
- Are tool calls still reliable?
- Did we introduce regressions?

Today, there is no universally accepted methodology for evaluating conversational AI systems.

This repository documents our research into the current state of AI evaluation.

Our goal is not to compare products.

Our goal is to understand the underlying principles that make AI systems reliable.

---

## Research Areas

- AI Observability
- LLM Evaluation
- Voice Agent Evaluation
- Conversation Benchmarking
- Prompt Experiments
- Regression Detection
- Synthetic Data Generation
- Production Monitoring
- AI CI/CD
- Agent Reliability

---

## Platforms Being Studied

| Player | Research status |
| --- | --- |
| [LangSmith](players/langsmith.md) | Earlier draft; revalidation pending |
| [Braintrust](players/braintrust.md) | Documentation reviewed |
| [Langfuse](players/langfuse.md) | Documentation reviewed |
| [OpenAI Evals](players/openai-evals.md) | Documentation reviewed; hosted-platform retirement recorded |
| [DeepEval](players/deepeval.md) | Earlier draft; revalidation pending |
| [Arize Phoenix](players/arize-phoenix.md) | Documentation reviewed |
| [Galileo](players/galileo.md) | Documentation reviewed |
| [Patronus AI](players/patronus.md) | Core guides reviewed; some catalog pages inaccessible |
| [Vapi](players/vapi.md) | Documentation reviewed |
| [Retell AI](players/retell.md) | Documentation reviewed |
| [Bland AI](players/bland.md) | Documentation reviewed |

---

## Research Philosophy

Before building software, understand the problem.

Instead of asking:

"What should we build?"

We ask:

"What problems still remain unsolved?"

---

Everything in this repository is part of an ongoing research effort.
