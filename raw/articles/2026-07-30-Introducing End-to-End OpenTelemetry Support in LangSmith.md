---
title: "Introducing End-to-End OpenTelemetry Support in LangSmith"
source_url: "https://www.langchain.com/blog/end-to-end-opentelemetry-langsmith"
ingested: 2026-07-30
blog: "LangChain Blog"
published: "2026-06-15"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-15
- **URL:** https://www.langchain.com/blog/end-to-end-opentelemetry-langsmith

## Article Content

Introducing End-to-End OpenTelemetry Support in LangSmith

LangSmith Platform

Agent Improvement

Engine

Improve agents autonomously

Observability

See exactly what your agents are doing

Evaluation

Score and improve agent performance

Agent Infrastructure

Deployment

Ship and scale agents in production

Sandboxes

Run agent-generated code safely

No-Code Agents

Fleet

Agents for the whole company

Open Source Frameworks

deepagents

Build long-running agents for complex tasks

langgraph

Build reliable agents with low-level control

langchain

Quick start agents with any model provider

Resources

Customer Stories

Guides

Max Agency

How-To

LangChain Academy

YouTube

Documentation

Community

LangSmith for Startups

Meetups

Community

About

Careers

Partners

Company Announcements

LangSmith

Introducing End-to-End OpenTelemetry Support in LangSmith

The LangChain Team

March 26, 2025

4

min

Create agents

Observability is critical for debugging and optimizing LLM applications — but until now, getting a complete view of your system meant juggling multiple tools and formats. Now, LangSmith offers full end-to-end OpenTelemetry support for applications built on LangChain and/or LangGraph.

With our OpenTelemetry (OTel) integration, you can standardize tracing across your stack and send traces to LangSmith — our testing & observability platform for the agent lifecycle — or other observability platforms.

Previously, LangSmith supported OpenTelemetry as only a backend trace ingestion format. With this update, we’re completing the picture by adding native OpenTelemetry support directly into the LangSmith SDK.

Why OpenTelemetry for LLM applications?

OpenTelemetry (OTel) is an open-source observability framework  that standardizes how telemetry data is collected, exported, and analyzed. As applications grow more complex and distributed, OpenTelemetry provides a consistent way to track performance, understand system behavior, and troubleshoot issues.

For LLM applications, observability presents unique challenges. Traditional application monitoring focuses on errors and compliance with expected behaviors — however, LLM observability requires understanding multi-step workflows and monitoring dynamic, stochastic outputs with complex evaluation metrics that go beyond simple error rates.

OpenTelemetry addresses these challenges by providing a unified, vendor-neutral standard for instrumentation that works across different languages, frameworks, and backends.

How our OpenTelemetry Pipeline Works

With this update, LangSmith now offers a complete OpenTelemetry pipeline for LLM applications:

LangChain instrumentation: Automatically generate detailed traces from your LangChain or LangGraph applications

LangSmith SDK: Convert and transport these traces through our SDK using OpenTelemetry's standardized format

LangSmith platform: Ingest and visualize traces in a powerful, LLM-specific observability dashboard

This end-to-end integration unlocks several key benefits:

Unified observability: View your entire application stack—from LangChain components to underlying infrastructure—in a single, cohesive view

Distributed tracing: Follow requests as they move through your microservices architecture, with context propagation ensuring that related spans are linked to the same trace

Interoperability: Connect LangSmith with your existing observability tools and infrastructure through the OpenTelemetry standard, including platforms like Datadog, Grafana, and Jaeger.

With this integration, you can trace the complete execution path of your LLM applications, from the initial prompt to the final response, with detailed visibility into each step along the way.

Getting Started with OpenTelemetry in LangSmith

1. Installation​

Install the LangSmith package with OpenTelemetry support:

pip install "langsmith[otel]"

pip install langchain

2. Enable the OpenTelemetry integration​

You can enable the OpenTelemetry integration by setting the LANGSMITH_OTEL_ENABLED environment variable:

LANGSMITH_OTEL_ENABLED=true

LANGSMITH_TRACING=true

LANGSMITH_ENDPOINT=https://api.smith.langchain.com

LANGSMITH_API_KEY=<your_langsmith_api_key>

3. Create a LangChain application with tracing​

Here's a simple example showing how to use the OpenTelemetry integration with LangChain:

import os

from langchain_openai import ChatOpenAI

from langchain_core.prompts import ChatPromptTemplate

# LangChain will automatically use OpenTelemetry to send traces to LangSmith

# because the LANGSMITH_OTEL_ENABLED environment variable is set

# Create a chain

prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")

model = ChatOpenAI()

chain = prompt | model

# Run the chain

result = chain.invoke({"topic": "programming"})

print(result.content)

4. View the traces in LangSmith​

Once your application runs, you'll see the traces in your LangSmith dashboard like this one.

Performance Considerations

While our end-to-end OpenTelemetry support provides maximum flexibility and interoperability, it comes with slightly higher overhead compared to LangSmith’s native tracing format.

For users that are exclusively using LangSmith as their observability platform, we still recommend our native tracing format for optimal performance. It offers realtime tracing with pending runs, faster ingest speeds, and reduced memory overhead from the sdk.

The native LangSmith tracing format has been specifically designed for LLM applications and offers several key advantages. It features significantly reduced overhead with a lower computational and memory footprint compared to the more general-purpose OpenTelemetry format. Our native format is also custom-tailored for the unique data patterns and volumes found in LLM applications.

Try it today

Ready to get started tracing your LangChain and LangGraph applications with OpenTelemetry? Check out our full documentation for more details and examples — and try out LangSmith for free if you haven't already.

Related content

Conceptual Guide

LangSmith

Building Governed Agents: A Framework for Cost, Control, and Compliance

Martha Janicki

July 20, 2026

15

min

Conceptual Guide

LangSmith

Agents need their own computer. Here's how to give them one safely.

Amy Ru

July 15, 2026

12

min

LangChain

LangSmith

Observability & Evals

Your coding agent bill doubled. Here’s how to fix it.

Amy Ru

July 2, 2026

6

min
