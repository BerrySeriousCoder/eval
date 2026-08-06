# DeepEval

Open-source LLM evaluation framework.

Built by the team at Confident AI.

Apache 2.0 licensed. Python-only.

GitHub: github.com/confident-ai/deepeval

They call it "Pytest for LLMs."


## Vision

Make LLM evaluation feel like unit testing.

You write test cases. You define metrics. You run them from the CLI.

If something regresses, you catch it before shipping.

The idea is that evaluation shouldn't be a separate workflow. It should live next to your application code, run in CI/CD, and give pass/fail results like any other test.

They want evals to be local-first, fast, and developer-owned.

Not a dashboard you check once a week.


## Users

Technical audience. Not for PMs clicking through a UI.

- AI engineers evaluating agents, RAG, tool calls
- Data scientists running prompt experiments
- QA teams writing regression tests for AI behavior
- Vibe coders using Claude Code, Cursor, Codex to build and iterate

They've been pushing hard on the "vibe coder" angle recently. The idea is you install a skill in Cursor or Claude Code and the coding agent writes the eval suite, runs it, and iterates on failures.

Used at Google, OpenAI, Microsoft according to their docs.

20 million+ daily evaluations claimed.


## Core abstraction

Everything in DeepEval revolves around FOUR things.

Test Case

    The atomic unit. A single interaction with your LLM.

    Has: input, actual_output, expected_output, context, retrieval_context, tools_called, expected_tools, token_cost, completion_time.

    Not all fields are required. Different metrics need different fields.

Golden

    A precursor to a test case. Basically a test case template without the actual_output.

    You store goldens in datasets. At evaluation time you run your LLM app against each golden to produce test cases.

Dataset (EvaluationDataset)

    A collection of goldens.

    Can be single-turn or multi-turn but not both.

    You can push/pull datasets to Confident AI cloud or save locally as JSON/CSV.

Metric

    The scoring logic. Takes a test case, produces a score between 0 and 1 plus a reason.

    A test case passes if every metric score >= threshold (default 0.5).

That's it. Those are the building blocks.


## Workflow

Typical workflow looks like this.

1. Define your dataset (goldens)

2. For each golden, run your LLM app to get actual_output

3. Create test cases from goldens + outputs

4. Run metrics against test cases

5. Get scores and pass/fail results

There are two ways to run evals.

deepeval test run (CLI, Pytest-based, good for CI/CD)

    You write a test file with assert_test()

    Run: deepeval test run test_file.py

    Results show up in terminal like pytest output

evaluate() (Python function, good for notebooks/scripts)

    Call evaluate(test_cases, metrics)

    Same results, no CLI needed

Both produce a "test run" — a snapshot of all test case results at that point in time.

If you're logged into Confident AI, results also get pushed to the cloud for dashboards and regression tracking.


## Architecture

Local-first.

Everything runs on your machine. No cloud required for core evaluation.

The flow is:

    Your LLM App
        ↓
    Test Case (input + actual_output)
        ↓
    Metric (uses an LLM judge to score)
        ↓
    Score + Reason + Pass/Fail

Metrics that use LLM-as-a-judge need an LLM API key. Default is OpenAI but you can swap in Anthropic, Gemini, Ollama, Azure OpenAI, DeepSeek, or any custom LLM.

For agents and complex apps they added tracing.

The @observe decorator captures function calls as spans. Spans nest to form a trace. You can attach metrics to individual spans so different components get evaluated with different metrics.

Component-level eval (trace-based):

    @observe()
    def my_agent(query):
        chunks = retrieve(query)       ← span
        answer = generate(query, chunks) ← span with metrics
        return answer

This is how you evaluate agents without flattening everything into a single test case.

Two modes of eval:

    End-to-end — treat app as black box. Input in, output out, score it.

    Component-level — trace the app, score individual spans.

You can combine both.


## SDK

Python only.

pip install deepeval

CLI tool: deepeval

Main commands:

    deepeval test run — run evaluations
    deepeval login — connect to Confident AI
    deepeval view — view results on platform
    deepeval inspect — view traces locally
    deepeval set-ollama / set-gemini / set-azure-openai — configure LLM provider

Key modules:

    deepeval.test_case — LLMTestCase, ConversationalTestCase, Turn, ToolCall
    deepeval.dataset — EvaluationDataset, Golden, ConversationalGolden
    deepeval.metrics — all built-in metrics
    deepeval.tracing — @observe, update_current_span, update_current_trace
    deepeval.synthesizer — Synthesizer for synthetic data generation
    deepeval.openai — drop-in OpenAI replacement that auto-traces

The Pytest integration is nice. You use @pytest.mark.parametrize to loop through test cases and assert_test() to pass/fail each one. It feels like writing regular tests.


## API

No REST API exposed by the open-source framework.

DeepEval is a local library. You import it and call functions.

The Confident AI platform has its own API for:
- pushing/pulling datasets
- viewing test runs
- managing projects

But the core DeepEval framework is just Python imports.


## Data model

Test Case (LLMTestCase)

    input: str — user input
    actual_output: str — LLM response
    expected_output: str — ideal response
    context: list[str] — ground truth context
    retrieval_context: list[str] — what RAG actually retrieved
    tools_called: list[ToolCall] — tools the agent used
    expected_tools: list[ToolCall] — tools it should have used
    token_cost: float
    completion_time: float

Conversational Test Case

    turns: list[Turn] — sequence of user/assistant messages
    scenario: str — description of the conversation context

Golden

    Same fields as test case but without actual_output.

    You add actual_output at evaluation time.

Conversational Golden

    scenario: str
    expected_outcome: str

Trace / Span

    Created by @observe decorator.
    Captures function inputs, outputs, duration.
    Spans can have metrics attached.

Test Run

    Collection of evaluated test cases.
    Timestamped snapshot.


## Metrics

This is where DeepEval really shines. 50+ metrics out of the box.

Custom (LLM-as-a-Judge)

    GEval — define criteria in natural language, it generates evaluation steps, scores with an LLM. Research-backed. This is the most flexible one.

    DAG (Deep Acyclic Graph) — decision-tree style evaluation. Deterministic. You define branching logic. Good for mixed objective/subjective criteria.

    ConversationalGEval — GEval but for multi-turn.

RAG Metrics

    Retriever:
        Contextual Relevancy — is the retrieved context relevant to the query?
        Contextual Precision — is the context focused?
        Contextual Recall — did we retrieve everything needed?

    Generator:
        Answer Relevancy — is the answer relevant to the input?
        Faithfulness — is the answer grounded in the retrieved context?

Agent Metrics

    Task Completion — did the agent finish the job? (needs tracing)
    Tool Correctness — did it call the right tools?
    Argument Correctness — did it pass the right arguments?
    Step Efficiency — did it take unnecessary steps?
    Plan Adherence — did it follow the plan?
    Plan Quality — was the plan good?

Conversational (Multi-turn)

    Knowledge Retention — does the chatbot remember earlier context?
    Role Adherence — does it stay in character?
    Conversation Completeness — did the conversation satisfy the user?
    Conversation Relevancy — are responses relevant to user turns?

Safety

    Bias
    Toxicity
    Non-Advice
    Misuse
    PII Leakage
    Role Violation

Multimodal

    Image Coherence
    Image Helpfulness
    Image Reference
    Text-to-Image
    Image-Editing

Other

    Hallucination
    JSON Correctness
    Summarization
    Ragas

All metrics score 0-1. All provide a reason.

Most use LLM-as-a-judge under the hood. You can swap the judge model per metric.


## Dashboard

DeepEval itself has no dashboard. It's a local CLI tool.

The dashboard lives on Confident AI (their commercial platform).

Confident AI gives you:

    - Sharable testing reports
    - Regression tracking (green = improved, red = regressed)
    - Trace visualization
    - Dataset management UI
    - Annotation queues for human review
    - Custom dashboards for stakeholders
    - Production monitoring and alerting

You connect DeepEval to Confident AI with:

    deepeval login

Then every test run automatically shows up on the platform.


## Integrations

Framework integrations (tracing):

    - LangChain
    - LangGraph
    - Pydantic AI
    - OpenAI SDK
    - OpenAI Agents SDK
    - Anthropic
    - CrewAI
    - LlamaIndex
    - Google ADK
    - AWS AgentCore
    - Strands Agents SDK

Evaluation model providers:

    - OpenAI
    - Azure OpenAI
    - Anthropic
    - Gemini
    - Ollama
    - DeepSeek
    - OpenRouter
    - Amazon Bedrock
    - Vertex AI
    - Grok
    - Moonshot
    - Portkey
    - vLLM
    - LM Studio
    - LiteLLM

Vector DBs:

    - Cognee
    - Elasticsearch
    - Chroma
    - Weaviate
    - Qdrant
    - PGVector

Other:

    - Hugging Face (training/eval callbacks)

Also part of the ecosystem:

    Confident AI — the commercial platform
    DeepTeam — red teaming / security testing framework (separate product)


## Pricing

DeepEval (the framework) — completely free. Apache 2.0.

Confident AI (the platform) — freemium.

    Free tier: limited test runs, basic features
    Paid tiers: more volume, longer retention, RBAC, custom dashboards, dedicated support
    Tracing: ~$1/GB-month for ingestion
    Enterprise: self-hosted, SSO, SOC2 Type II, GDPR, custom retention

Enterprise pricing is "book a demo."

The monetization model is classic open-source + commercial platform. Use DeepEval for free, pay for Confident AI when you need collaboration, observability, and production monitoring.


## What we like

Pytest-native approach is smart. Engineers already know how to write tests. Making evals feel like tests lowers the barrier.

50+ metrics out of the box is impressive. GEval is genuinely useful — you describe what you want to evaluate in plain English and it just works.

The DAG metric is interesting. Decision-tree-style LLM evaluation that's more deterministic than pure LLM-as-a-judge.

Local-first. No cloud dependency to run evals. You can run everything on your laptop.

Tracing with @observe is clean. Decorate a function, attach metrics, done. No need to restructure your code.

CLI is well-designed. deepeval test run is a memorable command.

Synthetic data generation built in. If you don't have test data, you can generate it from documents.

Active community. Maintainers are responsive. They actually fix things people complain about.

Good TypeScript/API coverage for common workflows even though it's Python only.

Research-backed metrics. Not just vibes — they reference actual papers (GEval, etc.)


## Weaknesses

Python only. No TypeScript SDK. If your stack is Node.js you're out of luck.

Heavy dependency on external LLM for evaluation. Every metric call is an LLM API call. That adds cost and latency.

LLM-as-a-judge metrics are inherently non-deterministic. Running the same eval twice can give different scores. They try to mitigate with DAG but the core issue remains.

No built-in production monitoring in the open-source version. You need Confident AI for that.

Not designed for real-time or online evaluation. This is a batch/offline tool. You can't run it inline with production traffic without significant overhead.

No native voice/audio evaluation. It's text-in, text-out. For voice agents, you'd need to handle STT/TTS evaluation separately.

The relationship between DeepEval (free) and Confident AI (paid) can be confusing. Some features you'd expect in the framework (like dashboards, regression views) only exist on the paid platform.

Synthesizer has been known to timeout on large datasets. Long-running generation can lose progress.


## User complaints

From Reddit and community:

- Dependency bloat was a major issue early on. They bundled large Hugging Face transformer libraries which made it hard to use in constrained environments like Lambda. They've since fixed this.

- Synthesizer timeouts. The synthetic data generator would hang on large jobs and lose all intermediate progress. Frustrating.

- Some confusion about where DeepEval ends and Confident AI begins. Features are split across both and it's not always clear what requires the paid platform.

- Not ideal for production observability. It's great for offline regression testing but if you need to monitor live agent loops at 3 AM, you probably need an observability platform like Langfuse or Arize Phoenix alongside it.

- Framework-driven vs. manual error analysis debate. Some people question whether running metrics automatically actually catches the right problems, vs. manually eyeballing outputs. The community consensus is they're complementary — DeepEval automates checks you discover through manual analysis.

Maintainers are notably responsive. They actively engage on Reddit and Discord and ship fixes quickly when issues are raised.


## What we'd improve

Add voice/audio-native evaluation. DeepEval doesn't handle STT/TTS quality, latency, interruption handling, or any of the voice-specific dimensions we care about. It's a text evaluation tool being used in a world that increasingly runs on voice.

Make the synthesizer more robust. Checkpoint intermediate results during long-running generation. Don't lose 45 minutes of work because of a timeout.

Build a TypeScript SDK. The AI ecosystem isn't Python-only anymore. LangChain has JS, Vercel AI SDK exists, a lot of agent infra is TypeScript.

Clearer separation between open-source and platform features. It's frustrating when you realize the feature you need requires Confident AI. Be upfront about it.

Add production-inline evaluation. The ability to score responses in real time (or near-real-time) as they happen, not just in batch after the fact.

For our use case specifically — we'd need to build a voice evaluation layer on top of DeepEval. The framework is solid for text but doesn't understand conversations as audio events. Latency, tone, interruptions, silence handling — none of that exists here.
