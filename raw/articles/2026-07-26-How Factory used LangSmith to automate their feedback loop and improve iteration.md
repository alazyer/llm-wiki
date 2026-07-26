---
title: "How Factory used LangSmith to automate their feedback loop and improve iteration speed by 2x"
source_url: "https://www.langchain.com/blog/customers-factory"
ingested: 2026-07-26
blog: "LangChain Blog"
published: "2026-06-16"
---

## Source Metadata

- **Blog:** LangChain Blog
- **URL:** https://www.langchain.com/blog/customers-factory
- **Fetched:** 2026-07-26
- **Extracted title:** How Factory used LangSmith to automate their feedback loop and improve iteration speed by 2x

## Article Content

How Factory used LangSmith to automate their feedback loop and improve iteration speed by 2x

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

Case Studies

LangSmith

Observability & Evals

How Factory used LangSmith to automate their feedback loop and improve iteration speed by 2x

The LangChain Team

June 19, 2024

5

min

Create agents

In today’s fast-paced software development, streamlined Software Development Lifecycle (SDLC) capabilities are essential. Factory is building the secure AI platform for SDLC automation. Factory’s fleet of Droids automate different stages of the SDLC, boosting engineering velocity for large organizations — and their Code Droid has achieved state-of-the-art performance in complex software development tasks. By leveraging Self-hosted LangSmith, Factory meets complex observability requirements for autonomous LLM systems while maintaining enterprise-level security and privacy.

Leveraging LangSmith for Secure and Reliable AI Operations

Self-hosted LangSmith provides the necessary observability infrastructure needed to manage complex LLM workflows while ensuring data privacy and security. Factory can deploy LangSmith into environments where tight data controls prevent most LLM infrastructure from operating successfully.

One primary challenge Factory faced was ensuring robust observability in their customers' environments. Traditional methods for tracking data flow across LLM pipelines and debugging context-awareness issues were cumbersome. Additionally, Factory’s custom LLM tooling made most LLM observability tools challenging to set-up. LangSmith offered a complete solution with custom tracing via a first-party API.

Factory integrated LangSmith to export traces to AWS CloudWatch logs, which allowed the team to precisely track data flow through various stages of the LLM pipeline. By linking LangSmith events and steps with CloudWatch logs, Factory’s engineers could pinpoint their position in the agentic stage. This integration helped maintain a single source of truth for data flow in LLM from one step to the next, which is mission-critical for debugging and optimization.

Main UI in LangSmith that Factory reviews in development.

Another challenge was debugging context-awareness issues in generated responses. Factory used LangSmith to link feedback directly to each LLM call, providing immediate insights into potential problems. This integration helped the team quickly identify and resolve issues like hallucinations without a proprietary logging system. With feedback available next to every LLM call, Factory could ensure that the AI’s outputs were contextually accurate and relevant based on real customer input.

Closing the Product Feedback Loop with LangSmith

In addition to observability, Factory used LangSmith to optimize product feedback loops, focusing on prompt optimization and feedback API utilization. Traditional methods of manual prompt optimization were time-consuming and often inaccurate. LangSmith’s Feedback API streamlined the process, enabling Factory to collect and analyze feedback, then refine their prompts based on real-time data.

Factory's feedback loop starts with the Droid posting a comment and collecting positive/negative feedback. LangSmith analyzes the data, then Factory's engineers use custom LangChain tooling to optimize the prompt, re-prompt the LLM, and improve accuracy and reduce errors.

Factory used the Feedback API to append feedback to various stages of their workflows. The feedback was then exported to datasets, and analyzed for patterns and areas for improvement.

By benchmarking examples and automating the optimization process, Factory increased their control over accuracy and enhanced the overall performance of their AI models. For example, with examples of comments on code with good or bad feedback, they had the LLM look at a prompt and make a claim as to why the prompt may have caused a bad example (and not a good example). This streamlined feedback collecting and processing not only improved prompt optimization but also reduced mental overhead and infrastructure requirements for analyzing feedback.

The ability to automate feedback collection and processing was particularly valuable for pipeline steps that required human feedback. With LangSmith, Factory could focus on the final stages of feedback collection, ensuring that the most critical aspects of their Droid’s performance were continually refined and optimized.

This approach led to significant improvements in accuracy and efficiency across their workflows. Compared to their previous method of manual data collection and human-driven prompt iteration, Factory was able to 2x their iteration speed. Factory also reports their average customer experienced a ~20% reduction in open-to-merge time, and a 3x reduction in code churn on code impacted by Droids in the first 90 days.

Looking Forward: Expanding AI Autonomy in the SDLC

As Factory continues to innovate, their focus remains on enhancing AI capabilities across the entire SDLC. Partnering with LangChain and using LangSmith have been pivotal in this journey, providing the tools and infrastructure needed to achieve unprecedented levels of efficiency and quality in software development.

Factory’s Droids have already led to remarkable improvements in engineering operations. Clients report an average reduction in cycle time by up to 20%, with over 550,000 hours of development time saved across various organizations. These substantial time savings allow engineering teams to focus on innovative, value-added tasks, enhancing overall productivity and reducing operational costs.

The future looks bright for Factory as they continue to push the boundaries of AI in software development. With the recent public launch of their AI Droids and $15 million in Series A funding led by Sequoia Capital, Factory is poised for significant growth and innovation. The ongoing collaboration with LangChain is a cornerstone of this strategy, ensuring that Factory remains at the cutting edge of AI-driven software development.

“Our collaboration with LangChain has been critical to successfully deploying enterprise LLM-based systems. We are significantly more confident in our decision making and operational capabilities thanks to the observability and orchestration-layer tooling that we get from the LangChain team.” – Eno Reyes, CTO of Factory

About Factory

Factory is an enterprise AI company dedicated to automating the software development lifecycle. By integrating advanced autonomous Droids, Factory helps businesses achieve faster, more reliable, and cost-effective software delivery.

For more insights and updates, visit Factory’s website.

About LangChain

LangChain, Inc. was founded in early 2023 to help developers build context-aware reasoning applications. The company’s popular open-source framework gives developers the building blocks to create production-ready applications with LLMs. LangSmith complements this as an all-in-one SaaS platform that enables a full, end-to-end development workflow for building and monitoring LangChain and LLM-powered apps.

For more information, visit LangChain’s website.

Related content

Open Source

Observability & Evals

How We Benchmark Deep Agents

Nick Hollon

Harrison Chase

July 23, 2026

4

min

Observability & Evals

Towards Automating Eval Engineering

Vivek Trivedy

July 22, 2026

5

min

Case Studies

How Apollo Rebuilt Its AI Assistant on Deep Agents to Power the Full GTM Loop

Sofia Sulikowski

July 21, 2026

6

min
