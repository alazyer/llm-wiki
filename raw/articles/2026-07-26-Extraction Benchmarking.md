---
title: "Extraction Benchmarking"
source_url: "https://www.langchain.com/blog/extraction-benchmarking"
ingested: 2026-07-26
blog: "LangChain Blog"
published: "2026-06-30"
---

## Source Metadata

- **Blog:** LangChain Blog
- **URL:** https://www.langchain.com/blog/extraction-benchmarking
- **Fetched:** 2026-07-26
- **Extracted title:** Extraction Benchmarking

## Article Content

Extraction Benchmarking

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

Extraction Benchmarking

Ankush Gola

December 5, 2023

27

min

Create agents

Two weeks ago, we launched the langchain-benchmarks package, along with a Q&A dataset over the LangChain docs. Today we’re releasing a new extraction dataset that measures LLMs' ability to infer the correct structured information from chat logs.

The new dataset offers a practical environment to test common challenges in LLM application development like classifying unstructured text, generating machine-readable information, and reasoning over multiple tasks with distracting information.

In the rest of this post, I'll walk through how we created the dataset and share some initial benchmark results. We hope you find this useful for your own conversational app development and would love your feedback!

Selected metric comparison

Motivation for the dataset

We wanted to design the dataset schema around a real-world problem: gleaning structured insights from chat bot interactions.

Over the summer, our excellent intern Molly helped us refresh Chat LangChain (repo), a retrieval-augmented generation (RAG) application over LangChain's python docs. It’s an “LLM with a search engine”, so you can ask it questions like "How do I add memory to an agent?”, and it will tell you an answer based on whatever it can find in the docs.

The real test of such a project begins post-deployment, when you begin to observe how it's used and refine it further. Typically, users won't provide explicit feedback, but their conversations reveal a lot, and while you can try just “putting the logs into an LLM” to summarize it, you can also often benefit from extracting structured content to monitor and analyze. This could help drive analytic dashboards or fine-tuning data collection pipelines, since the structured values can easily be used by traditional software.

The Chat Extraction dataset is designed around testing how well today's crop of LLMs are able to extract and categorize relevant information from this type of data.  In the following section, I’ll walk through how we created the dataset. If you just want to see the results, check out the summary graph below. You can feel free to jump to the experiments section for an analysis of the results.

Screenshot of the benchmark results

Creating the Dataset

The main steps for creating the dataset were:

Settle on a data model to represent the structured output.

Seed with Q&A pairs.

Generate candidate answers using an LLM.

Manually review the results in the annotation queue, updating the taxonomy where necessary.

LangChain has long had synthetic dataset generation utilities that help you bootstrap some initial data, but the final version should always involve some amount of human review to ensure proper quality. That’s why we’ve added data annotation queue’s to LangSmith and will continue to improve our tooling to help you build your data flywheel.

Once you have an initial dataset, you can use the labeled data as few-shot examples within the seed-generation model to improve the quality of data given to humans for review. This can help reduce the amount of work and changes needed when updating the ground truth.

Extraction Schema

We wanted the task to be tractable while still offering a challenge for many common models today. We defined the schema using this linked pydantic model. An example extracted value is below:

"GenerateTicket": {

"question": {

"toxicity": 0,

"sentiment": "Neutral",

"is_off_topic": false,

"question_category": "Function Calling",

"programming_language": "unknown"

"response": {

"response_type": "provide guidance",

"confidence_level": 5,

"followup_actions": [

"Check with API provider for function calling support."

"issue_summary": "Function Calling Format Validation"

Example Extracted Output

Many of these values could be useful in monitoring an actual production chat bot. We made the schema challenging in a few ways to make the benchmark results more useful in separating model capacity and functionality. Some challenges about this schema include:

It includes a couple fairly long Enum values. Even OpenAI's function calling/tool usage API can be imperfect in generating these.

The object is nested - nesting can make it harder for LLMs to stay coherent if they aren't trained on code.

The values in each nested component are meant to be inferred only from the corresponding sections of input (response or question).

It combines classification, summarization, and structured output generation in a single task.

If "attention is all you need", by splitting the attention of the model, this multi-task objective can be challenging for an LLM to address in a single generation.

Evaluation

This benchmark is focused on structure and classification, and as such, we don't need to use any LLM-as-a-judge metrics. Instead, we wrote custom LangSmith evaluators (see the code definition here). Below is what we measured:

Structure verification

json_schema : 1 if correct, 0 if not. We validate the parsed output for each model using the task schema.

Classification tasks

question_category: classification accuracy over the 25 valid enum values.

off_topic_similarity: binary classification accuracy of whether the LLM considered the question off-topic

toxicity_similarity: normalized difference in predicted level of "toxicity" of the user question.

programming_language_similarity - classification accuracy of the predicted programming language the user's question references. In most cases, this is "unknown".

confidence_level_similarity the normalized similarity between the predicted "confidence" of the response and the labeled confidence.

sentiment_similarity - Normalized difference between the prediction and label. Sentiment is scored as 0/1/2 for negative/neutral/positive.

Overall difference

json_edit_distance: this is a bit of a catch-all that first canonicalizes the predicted json and label json and then computes the Damerau-Levenshtein string distance between the two serialized forms.

Experiments

In making this dataset, we wanted to answer a few questions:
