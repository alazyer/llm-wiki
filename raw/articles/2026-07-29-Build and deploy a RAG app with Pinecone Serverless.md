---
title: "Build and deploy a RAG app with Pinecone Serverless"
source_url: "https://www.langchain.com/blog/pinecone-serverless"
ingested: 2026-07-29
blog: "LangChain Blog"
published: "2026-06-15"
---
## Source Metadata
- **Blog:** LangChain Blog
- **Published:** 2026-06-15
- **URL:** https://www.langchain.com/blog/pinecone-serverless

## Article Content

Build and deploy a RAG app with Pinecone Serverless

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

Tutorials & How-Tos

Build and deploy a RAG app with Pinecone Serverless

The LangChain Team

January 16, 2024

4

min

Create agents

Key Links

Repo: https://github.com/langchain-ai/pinecone-serverless

Video: https://youtu.be/EhlPDL4QrWY

Context

LLMs are unlocking a new era of generative AI applications, becoming the kernel process of a new kind of operating system. Just as modern computers have RAM and file access, LLMs have a context window that can be loaded with information retrieved from external data sources, such as databases or vectorstores.

Retrieved information can be loaded into the context window and used in LLM output generation, a process called retrieval augmented generation (RAG). RAG is a central concept in LLM app development because it can reduce hallucinations by grounding output and adds context that is not present the training data.

Challenges with production

With these points in mind, vectorstores have gained considerable popularity in production RAG applications because they offer a good way to store and retrieve relevant context. In particular, semantic similarity search is commonly used to retrieve chunks of information that are relevant to a user-provided input.

A large number of RAG demos have been shared over the past months, often using tools such as Jupyter notebooks and local vectorstores. Yet, several pain points create a gap between these demos and production RAG applications. Below, we'll discuss several ways to overcome these gaps and provide both a repo and a hands-on video that builds a production RAG application from scratch.

Pain Point

Detail

Solutions

Hosted vectorstore management

Usage-based-pricing and unlimited scalability

Pinecone serverless

Rapid RAG application deployment

Rapid deployment of prototype RAG applications

Hosted LangServe

RAG observability

Seamless observability of the RAG application

LangSmith

Support for production

Pinecone Serverless
