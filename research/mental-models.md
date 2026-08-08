## Insight #1

DeepEval treats evaluation like unit testing.

Everything begins with a Test Case.

This makes it excellent for CI/CD.

Tradeoff:

Weak production observability.

---

## Insight #2

DeepEval separates

Golden

↓

Test Case

This allows datasets to stay reusable
while actual outputs change every run.

---

## Insight #3

LangSmith is observability-first, not test-first.

Everything begins with a Trace.

App Execution

↓

Trace (Nested Runs)

↓

Observe / Evaluate

Evaluation sits on top of tracing.

Tradeoff:

SaaS-heavy and cost escalates with high production traffic.

---

## Insight #4

LangSmith connects production directly to evaluation (Data Flywheel).

Production Trace

↓

Flagged Edge Case

↓

Add to Dataset (1-Click)

↓

Offline Regression Test

Datasets are not static — they grow from real user failures.

---

## Insight #5

LangSmith separates Thread from Trace.

Thread = Entire Conversation / Voice Call

Trace = Single Turn / Nested Application Execution

This allows evaluating both individual turn latency and overall conversation outcomes.