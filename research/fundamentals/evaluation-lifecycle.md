# Step 1 : Define What You're Testing

First question.

    What changed?

Examples

    Prompt v15

        ↓

    Prompt v16

or

    GPT-5.5
    
        ↓
    
    Claude 5

or

    Old Voice Agent

        ↓

    New Voice Agent

This becomes

System Under Test (SUT)

# Step 2 — Select Evaluation Dataset

You need conversations.

Not random ones.

Representative ones.

Example

                            Appointment Booking

                            Refund

                            Sales Call

                            Customer Angry

                            Customer Interrupts

                            Customer Changes Mind

                            Bad Audio

                            Accent

                            Background Noise

This collection is called

>Evaluation Dataset

There are actually THREE main sources of evaluation data.

1. Synthetic

   Generated conversations

2. Benchmark Dataset

   Curated scenarios

3. Production Replay

   Real customer conversations

Not all evaluations begin the same way.


# Step 3 — Execute

Every scenario is executed.

    Scenario 1
    
    ↓
    
    Run Conversation
    
    ↓
    
    Collect Everything

Collect 

                Transcript

                Tool Calls

                Latency

                Tokens

                Errors

                Audio

                Logs

                Cost

execution isn't always the same.

Chat Agent, Voice Agent, Coding agent every agent has there own execution steps.

Execution depends on SUT (System Under Test)

# Step 4 — Evaluate

This is where different companies differ.

Some use

Rule Engine

Some use

LLM Judge

Some use

Humans

Some use

Business Metrics

We'll study that later.

# Step 5 — Aggregate

Now we have

1000 conversations.

Each has

scores.

Need one report.

Example

            Accuracy
            
            94%
            
            Latency
            
            310ms
            
            Hallucinations
            
            2%
            
            Booking Success
            
            97%

# Step 6 — Compare

Compare

Old vs New

        Booking Success

        Old

        92%

        New

        95%

This tells whether the change improved things.


What does every Evaluation Framework fundamentally need .

I think it's only five thinks.

        System Under Test

        Scenario

        Execution

        Evaluator

        Result

