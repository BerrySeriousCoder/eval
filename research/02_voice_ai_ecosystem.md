# What is a Voice Agent?

A voice agent is an AI system capable of interacting with humans through natural spoken conversation.

Unlike traditional chatbots, a voice agent operates on streaming audio and must process user speech, reason about the conversation, interact with external systems, and generate natural spoken responses all under strict latency constraints.

# High-Level Architecture

                Human

                  │

           Speech (Audio)

                  │

      Speech-to-Text (STT)

                  │

      Conversation Engine

     ├── LLM
     ├── Prompt
     ├── Memory
     ├── Tools
     ├── Business Logic
     ├── RAG
     └── Workflow

                  │

      Text Response

                  │

      Text-to-Speech (TTS)

                  │

           Audio Response

                  │

                Human


# Components

# 1. Speech-to-Text

Converts audio into text.

Examples

- Deepgram
- AssemblyAI
- Gladia
- Whisper
- Speechmatics

Evaluation

- Word Error Rate
- Number Accuracy
- Proper Noun Accuracy
- Accent Robustness
- Latency


# 2. Conversation Engine

The "brain."

Responsible for

- understanding intent
- reasoning
- planning
- memory
- tool calling
- workflow execution

This is where most existing LLM evaluation frameworks focus.

# 3. Tool Layer

The agent interacts with external systems.

Examples

- CRM
- Calendar
- Stripe
- Database
- Internal APIs

Failures here directly affect business outcomes.

# 4. Text-to-Speech

Generates spoken responses.

Examples

- ElevenLabs
- Cartesia
- PlayHT

Evaluation

- latency
- naturalness
- pronunciation
- interruptions
- emotion

# End-to-End Goal

A production voice agent isn't judged by how good its transcript is.

It's judged by whether it successfully completes the user's objective.

Examples:

- Appointment booked
- Refund processed
- Lead qualified
- Support ticket resolved
- Payment collected

Business outcome is the ultimate metric.