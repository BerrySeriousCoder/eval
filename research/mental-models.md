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