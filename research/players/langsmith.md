# LangSmith

Agent engineering and observability platform.

Built by the team at LangChain.

SaaS + Enterprise self-hosted. Multi-language SDKs.

Website: smith.langchain.com


## Vision

Make building, observing, evaluating, and shipping AI agents reliable.

Its core philosophy is not simply "run evaluations."

Instead, LangSmith focuses on the complete agent engineering flywheel:

    Build
      ↓
    Observe
      ↓
    Evaluate
      ↓
    Deploy
      ↓
    Monitor
      ↓
    Learn from production
      ↓
    Improve
      ↓
    Repeat

The central idea is that AI applications are hard to debug because execution is non-deterministic and multi-step (LLM calls, tool calls, retrieval, branching logic, multi-turn loops).

LangSmith places tracing at the center of everything. You instrument your application first to get complete visibility.

Evaluation then sits on top of these executions both offline using datasets before deployment, and online using live production traces.

The ultimate goal is closing the loop between production behavior and evaluation:

    Production Traces
        ↓
    Identify Failures / Edge Cases
        ↓
    Turn Traces into Evaluation Goldens
        ↓
    Run Offline Evals
        ↓
    Improve Agent
        ↓
    Deploy Again


## Users

Targeted at software engineers, AI engineers, and enterprise teams building production LLM applications and agents.

- AI engineers building agents, multi-step chains, RAG pipelines
- ML engineers evaluating prompt changes, model migrations, temperature tweaks
- Enterprise AI teams needing security, audit logs, and collaboration
- Product managers inspecting traces, annotating responses, and curating datasets
- Teams using LangChain or LangGraph
- Teams using custom frameworks, OpenAI SDK, AutoGen, CrewAI, Pydantic AI

LangSmith is framework-agnostic. While it integrates out-of-the-box with LangChain, you can use it with any language or custom architecture.

Used by Elastic, Klarna, Moody's, Uber, Replit, and thousands of AI teams.


## Core abstraction

Everything in LangSmith revolves around FIVE core building blocks.

Run

    The atomic execution unit. Represents a single step in an application (LLM call, retriever, tool execution, chain).

    Has: inputs, outputs, error, metadata, start_time, end_time, parent_run_id, trace_id.

Trace

    A tree of nested Runs representing an end-to-end execution of an application.

    Conceptually:

        Agent Run (Root)
        ├── Retriever Run
        ├── LLM Call Run
        ├── Tool Call Run (e.g. search_db)
        └── LLM Call Run (Final Answer)

Thread (Session)

    A collection of Traces grouped over time into a multi-turn conversation or user session.

    For a voice agent, a Thread represents an entire phone call or voice session, while individual turn interactions are Traces inside that Thread.

Dataset (and Examples)

    A collection of test inputs and expected outputs (Examples) used for offline evaluation.

    Examples can be uploaded from CSV/JSON, created programmatically, or added with one click from interesting production traces.

Evaluator

    The scoring logic. Can be a Code evaluator (exact match, regex, custom Python/JS function), an LLM-as-a-Judge, or a Human annotation.

    Operates on a Run/Trace and optional Example output, producing a score (0 to 1) plus feedback comments.


## Workflow

LangSmith supports three primary operational flows.

Offline Evaluation (Pre-deployment)

    You run your agent against a curated dataset to catch regressions before shipping.

    The flow is:

        Dataset (Goldens/Examples)
            ↓
        Run Target Application / Agent
            ↓
        Capture Traces & Outputs
            ↓
        Run Evaluators (Code + LLM-as-a-Judge)
            ↓
        Generate Experiment Results
            ↓
        Compare Version A vs Version B

Online Evaluation (Post-deployment)

    You monitor live production traffic by scoring real traces automatically.

    The flow is:

        Real User Interaction
            ↓
        Production Agent Execution
            ↓
        Stream Trace to LangSmith
            ↓
        Online Evaluator (LLM Judge / Rule)
            ↓
        Record Feedback & Score
            ↓
        Trigger Alerts / Dashboard Update

Production → Evaluation Feedback Loop (Data Flywheel)

    Turn real-world edge cases into permanent test cases.

    The flow is:

        Production Trace
            ↓
        Flagged Failure / Edge Case
            ↓
        Human Review / Annotation
            ↓
        "Add to Dataset" (1-Click)
            ↓
        New Evaluation Example Created
            ↓
        Regression Suite Enriched


## Architecture

SaaS platform backed by multi-language SDKs and OpenTelemetry.

The flow is:

    Your Application (Python / JS / Go / Java)
        ↓  (Instrumentation via @traceable / OTel)
    Async Background Exporter
        ↓
    LangSmith Ingestion Backend
        ↓
    Storage (Traces, Runs, Datasets, Feedback)
        ↓
    ┌─────────────────────────┬─────────────────────────┐
    ▼                         ▼                         ▼
  Observability Layer      Evaluation Layer        LangSmith Engine
  (Trace Explorer,         (Offline Experiments,   (Failure Clustering,
   Waterfall Timelines,     Online Evaluators,      Root Cause Analysis,
   Latency/Cost graphs)     Human Annotations)      Auto-remediation)

Tracing is asynchronous and non-blocking. SDKs batch trace runs in a background thread/task to avoid adding latency to application code.

Two main evaluation modes:

    Dataset-based — evaluate your agent against fixed test cases in batch.

    Trace-based — attach evaluators directly to production trace streams.


## SDK

Multi-language support. Official SDKs for Python, TypeScript/JavaScript, Go, and Java.

Python:

    pip install langsmith

TypeScript:

    npm install langsmith

Instrumentation is clean. In Python, use `@traceable`:

    from langsmith import traceable

    @traceable
    def my_agent(user_input: str):
        response = llm.invoke(user_input)
        return response

In TypeScript, wrap functions with `traceable()`:

    import { traceable } from "langsmith/traceable";

    const myAgent = traceable(async (input: string) => {
      // agent logic
    });

Key SDK modules and capabilities:

    Client — dataset management, feedback submission, experiment tracking
    evaluate() — helper function to run offline dataset evaluations
    @traceable / traceable() — function wrappers for nested trace capture
    OpenTelemetry Exporter — standard OTel exporter for custom enterprise pipelines

Native auto-tracing built into LangChain and LangGraph via environment variables:

    LANGCHAIN_TRACING_V2=true
    LANGCHAIN_API_KEY=ls__...


## API

Full REST API and GraphQL endpoints exposed.

The SDKs are thin wrappers over the REST API.

Key API endpoints cover:

    /runs — ingest and query execution runs/traces
    /datasets — create, update, list datasets and examples
    /experiments — submit evaluation runs and fetch aggregate scores
    /feedback — attach human or automated feedback to specific runs
    /prompts — fetch and push prompt versions from Prompt Hub

Allows headless integration into CI/CD pipelines (GitHub Actions, GitLab CI), webhooks on evaluation failure, and custom backend dashboards.


## Data model

Run

    id: UUID — unique run identifier
    name: str — function/component name (e.g. "ChatOpenAI", "retriever")
    run_type: str — type ('llm', 'chain', 'tool', 'retriever', 'prompt', 'parser')
    inputs: dict — input parameters / prompt string / messages
    outputs: dict — output payload / completion string / tool results
    error: str | None — exception trace if step failed
    start_time: datetime
    end_time: datetime
    extra: dict — metadata (tags, model params, latency, token costs)
    parent_run_id: UUID | None — parent run link for nesting
    trace_id: UUID — root run ID of the trace tree
    session_id: UUID | None — thread ID linking multi-turn runs

Trace

    Root Run object + tree hierarchy of child Runs joined by parent_run_id.

Thread (Session)

    session_id: UUID
    name: str
    metadata: dict — user ID, session metadata

Dataset

    id: UUID
    name: str
    description: str
    data_type: str — 'kv' (key-value), 'llm' (prompt/completion), 'chat' (messages)

Example

    id: UUID
    dataset_id: UUID
    inputs: dict
    outputs: dict
    metadata: dict

Feedback

    id: UUID
    run_id: UUID
    key: str — metric name (e.g. "correctness", "latency_pass")
    score: float | int | bool | None — numerical or boolean score
    value: str | None — text explanation or categorical label
    comment: str | None — human reviewer note
    source_type: str — 'api', 'model' (LLM judge), 'human'


## Metrics

Supports code-based checks, LLM-as-a-Judge evaluators, and human annotation queues.

Code Evaluators (Deterministic)

    Exact Match — binary check if actual output matches reference output.
    Regex Matching — verify formatting or structural patterns (e.g., JSON schema).
    String Distance — Levenshtein, BLEU, ROUGE scores.
    Custom Python/JS Functions — write arbitrary code returning a score dictionary.

LLM-as-a-Judge Evaluators (Subjective)

    QA / Correctness — checks if output accurately answers input given reference ground truth.
    Hallucination / Faithfulness — verifies output is grounded in retrieved context.
    Context Relevancy — evaluates retriever precision for RAG workflows.
    Criteria Evaluators — evaluate arbitrary criteria defined in natural language rubrics (e.g., conciseness, politeness, safety).
    Pairwise Evaluation — present output from Agent A and Agent B side-by-side to an LLM judge to determine winner.

Human Evaluation & Annotation

    In-UI review queue for human annotators.
    Thumbs up/down feedback buttons in trace view.
    Custom annotation schemas (multi-select, scale 1-5, free text).


## Dashboard

Rich SaaS interface for debugging, dataset management, and production monitoring.

Trace Explorer (Waterfall View)

    Interactive execution tree showing every nested run.
    Timeline waterfall breaking down latency per step (LLM generation, tool call, retrieval).
    Inspect token counts, exact prompts, raw API request/response payloads, and errors.

Datasets & Testing View

    Side-by-side experiment comparison tables (Experiment v1 vs Experiment v2).
    Diff view comparing actual output vs expected output across dataset rows.
    Aggregate score summary cards (pass rate, average correctness, latency p90).

Production Monitoring

    Time-series charts for latency distribution (p50, p90, p99), token throughput, cost estimation, and error rate.
    Online feedback metric tracking over time.

Prompt Hub & Playground

    Test, version, and collaborate on prompts directly in UI.
    Pull prompt versions into code via SDK and run evals directly against playground prompts.


## Integrations

Agent Frameworks:

    - LangChain
    - LangGraph
    - OpenAI SDK / Assistants API
    - AutoGen
    - CrewAI
    - LlamaIndex
    - Pydantic AI
    - Vercel AI SDK

Model Providers:

    - OpenAI
    - Anthropic
    - Google Gemini
    - AWS Bedrock
    - Azure OpenAI
    - Ollama
    - Groq
    - Together AI
    - LiteLLM

Observability & Infra:

    - OpenTelemetry (OTel exporter)
    - Datadog (via webhook / export)
    - Slack (alerts on failure)


## Pricing

Freemium SaaS model with usage-based billing and Enterprise self-hosted tiers.

Developer (Free Tier)

    - 5,000 free trace seats / month
    - Basic dataset management
    - 14-day trace retention
    - Single user

Plus Tier ($39/user/month base)

    - Pay-as-you-go trace usage (~$0.50 per 1,000 trace runs)
    - 400-day trace retention
    - Online evaluators & automated queues
    - API access & unlimited datasets

Enterprise Tier (Custom / Contact Sales)

    - Self-hosted deployment (AWS / GCP / Azure Kubernetes) or dedicated tenant
    - Custom trace retention
    - SOC2 Type II, HIPAA compliance, SAML/SSO
    - Custom rate limits and dedicated support


## What we like

Tracing-first model is superior for multi-step LLM agents and voice workflows where single-turn text evaluation fails to capture hidden step breakdowns.

The production flywheel (turning bad live traces into dataset test cases with 1-click) is the gold standard for continuous agent improvement.

Multi-language SDK support (Python, TypeScript, Go, Java) plus OpenTelemetry export means it fits into polyglot enterprise stacks.

Thread / Session abstraction naturally handles multi-turn conversation loops.

The waterfall trace visualizer is best-in-class for pinpointing latency bottlenecks in complex chains.

Prompt Hub integrated directly with tracing and testing makes prompt iteration fast.


## Weaknesses

SaaS / Closed Platform core — while SDKs are open-source, the full UI, dashboard, and dataset comparison backend are proprietary commercial SaaS (unlike open-source Langfuse or local DeepEval).

Trace cost can escalate rapidly on high-throughput production applications if sampling rate is not configured.

Not designed specifically for Voice AI — lacks out-of-the-box voice metrics (audio latency breakdown, STT WER, TTS voice quality, interruption/barge-in detection).

Can feel overly complex and heavyweight for simple single-prompt text apps.

Self-hosting is restricted to Enterprise plan contracts.


## User complaints

From Reddit, Discord, and GitHub community:

- High cost at scale: volume pricing on production traces can get expensive fast for high-volume apps.

- UI complexity: the interface has many deep sub-menus and dense information hierarchy, creating a learning curve for non-LangChain users.

- Vendor lock-in perception: although framework-agnostic, the platform is heavily optimized around LangChain/LangGraph idioms, which can make non-LangChain users feel like second-class citizens.

- Complex self-hosting setup: self-hosting is only supported via enterprise license and k8s helm charts, with no lightweight docker-compose single-command option for small teams.


## What we'd improve

Add native Voice AI evaluation primitives: audio latency (STT + LLM + TTS), STT Word Error Rate (WER), turn-taking silence gap detection, and barge-in / interruption handling.

Provide a lightweight open-source local viewer (like Langfuse or local DeepEval) for local offline trace inspection without needing a cloud account.

Add audio waveform rendering in the trace visualizer for voice agent runs so engineers can listen to exact audio spans alongside text transcripts.

Offer smarter default sampling rules to prevent runaway trace ingestion costs on high-volume production streams.

Simplify the UI navigation specifically for evaluation experiments vs observability traces.