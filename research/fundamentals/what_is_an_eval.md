# What is an evaluation?

>An evaluation is the process of measuring whether a system behaves as expected under a given set of conditions.


Every evaluation has exactly four things.

                                Input

                                ↓

                                System Under Test (SUT)

                                ↓

                                Observed Behaviour

                                ↓

                                Judgement


# What exactly can be evaluated?

                 AI System

                     │

    ┌────────────────┼─────────────────┐

                    Prompt

                    LLM

                    Memory

                    RAG

                    Tool Calling

                    Business Logic

                    Workflow

                    STT

                    TTS

                    Entire Pipeline


Anything that can change can be evaluated.

# What is NOT an evaluation?

Example

        Latency

        250ms

Is that an evaluation?

No.

It's a metric.

Evaluation would be

        Latency

        250ms

        Requirement

        <500ms

        PASS

Very different.

