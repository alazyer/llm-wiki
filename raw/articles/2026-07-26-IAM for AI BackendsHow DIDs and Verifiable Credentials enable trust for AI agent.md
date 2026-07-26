---
title: "IAM for AI BackendsHow DIDs and Verifiable Credentials enable trust for AI agentsDec 5, 2024 · 27 min"
source_url: "https://agentfield.ai/blog/iam-ai-backends"
ingested: 2026-07-26
blog: "AgentField"
published: "2026-07-24"
---

## Source Metadata

- **Blog:** AgentField
- **URL:** https://agentfield.ai/blog/iam-ai-backends
- **Fetched:** 2026-07-26
- **Extracted title:** IAM for AI Backends

## Article Content

IAM for AI Backends — AgentField

Skip to content

agentfield.ai

DevelopersExamplesIntegrationsEnterpriseBlog

Docs......Star...

Save for later

Blog · December 5, 2024

IAM for AI Backends

How DIDs and Verifiable Credentials enable trust for AI agents

Santosh Kumar Radha·Co-founder & CTO·

27 min read

Read this later

Send to inbox →

We'll send this piece + the next one we publish. No spam. Unsubscribe in one click.

In The AI Backend, we argued that your backend needs a reasoning layer. Not a chatbot, not a copilot, but a layer that sits alongside your services, making decisions that used to be hardcoded.

We called the principle "guided autonomy": software that reasons freely within boundaries you define, predictable enough to trust, flexible enough to be useful.

But here's what we didn't address: how do you actually trust it?

When software executes predetermined logic, trust is straightforward. You wrote the code, you know what it does, and the behavior is bounded by the instructions you gave it. Logs work great.

When software reasons, trust becomes a problem. It makes decisions based on context that didn't exist when you wrote the code, delegates to other agents, and acts on behalf of users who never explicitly approved each action.

If you can't answer "who did this, and were they authorized?" then you don't have guided autonomy. You have unaccountable autonomy, and that's a liability, not a feature.

Beyond the API Key

The current conversation around AI security is too narrow, focused on prompt injection, jailbreaking, and model safety. These are real, and they have value, but they miss the larger shift.

Think about your identity infrastructure. It has layers: authentication to prove who you are, authorization to control what you can do, permissions to scope access, audit to track what happened. Each layer assumes something fundamental: the client is predictable.

Your payment service doesn't suddenly decide to call an unexpected endpoint. Your notification service doesn't reason its way into customer data it wasn't designed to touch. The code does what the code does.

API keys and OAuth were built for this world. One key equals one identity equals one set of permissions. The protocol doesn't need to distinguish between "the user wanted this" and "the software decided this" because deterministic code naturally enforces fixed workflows.

Something fundamental is absent from this stack.

An identity layer designed for software that reasons. One that doesn't assume predictable behavior. One where delegation is a first-class primitive. One where every action carries cryptographic proof of who authorized it, why, and under what constraints.

Your reasoning layer needs identity infrastructure natively built for autonomous reasoners: infrastructure that doesn't assume bounded behavior the way API keys do, that doesn't conflate user intent with client action the way OAuth tokens do, that doesn't depend on someone telling the truth the way audit logs do.

Identity infrastructure designed for how autonomous software actually operates.

The Authorization Paradox

— A pause

Want the rest in your inbox?

We'll send you this article so you can finish it later — plus the next thing we publish. One click to unsubscribe.

Send it

The approaches dominating today both fail, and for related reasons.

The callback trap. Require authorization checks at every decision point. Every time an agent acts, call back to an auth server.

Consider what happens when you ask an agent to book a trip. A human searches sequentially, checking one flight site, picking dates, checking one hotel site, comparing prices, and repeating the cycle. Maybe 20-30 requests over an hour.

An agent operates differently. It can fan out in parallel: query Kayak, Expedia, Google Flights, and Skyscanner simultaneously for every date combination in your window. Each flight result triggers parallel hotel searches across Booking.com, Hotels.com, Airbnb. Each hotel triggers car rental and activity searches. One user request cascades into hundreds or thousands of parallel API calls.

The power of autonomous software, and exactly where callback-based auth breaks down.

The problem isn't rate limits or infrastructure capacity. The problem is architectural. Callback-based authorization assumes a synchronous, centralized model: every action requires a round-trip to a trusted authority. This worked when the "client" was a human clicking buttons, generating maybe a few requests per minute. Agents operate at machine speed, generating orders of magnitude more decisions, and each decision blocked on a network call defeats the purpose.

More fundamentally, callback models can't express what agents actually need. As recent research notes, "OAuth scopes are predefined and static; they cannot express lineage (which agent invoked whom) or context (why this action occurred)"[1]. The authorization check happens, but it answers the wrong question. It verifies that a token is valid, not that this specific action by this specific agent in this specific context was authorized.

The broad permission fantasy. Give agents sweeping access to avoid the bottleneck. Issue tokens with wide scopes and trust the agent to stay within bounds. The pattern spreading fastest right now, it demos beautifully but also fails catastrophically in production. You can't audit what you can't attribute, and you can't revoke what you can't scope. Accountability vanishes, and you're back to "trust us" security, which isn't security at all.

A better path exists: identity infrastructure that enables delegation, local verification, and cryptographic proof of every action. We call this accountable autonomy.

The answer isn't callbacks at every hop, and it isn't blind trust in broad permissions. Identity infrastructure where every agent has a real identity, every delegation is scoped, and every action is signed. Guided autonomy requires accountable identity.

What Autonomous Software Actually Needs

Building software that can think requires new primitives for trust. Not features you bolt on, but infrastructure that exists from the first line of code.

Identity that is real, not assumed.

When your payment service calls your notification service, you know who's calling because you deployed both. The identity is implicit in the architecture.

When your reasoning agent spawns a sub-agent that delegates to another sub-agent that calls an external service, who is calling? The answer can't be "whoever holds this token." It has to be cryptographically verifiable.

Each agent needs its own identity. Not a shared service account, not a forwarded token, but a real, cryptographic identity that can be independently verified.

Authorization that can delegate.

Traditional auth is binary: you have access or you don't. Agents need something else, the ability to grant scoped, time-limited authority to other agents, authority that attenuates with each delegation narrower than the last.

When Agent A delegates to Agent B, Agent B shouldn't get Agent A's full permissions. It should get exactly what it needs for this specific task, nothing more.

Permissions that travel with the request.

OAuth permissions live in the auth server, and you check them by calling back. But what happens when the request crosses organizational boundaries, when your agent talks to a partner's agent?

Permissions need to be portable, verifiable without callbacks, and cryptographically bound to the request itself.

Audit that is proof, not logs.

Logs record what someone claims happened. They can be edited, they can be incomplete, and they're not proof. They're a record that someone promises is accurate.

When an agent makes a decision, the audit trail needs to be cryptographic, tamper-proof, and independently verifiable, years later, on an air-gapped machine, without trusting anyone's servers.

These aren't nice-to-haves. They're architectural requirements of autonomous software. You can bolt security onto deterministic code, but you can't bolt accountability onto reasoning systems.

We built AgentField with identity, authorization, permissions, and audit from day one.

Why OAuth Can't Get You There

OAuth was designed for a different problem. The core assumption is that "the client's request embodies the resource owner's intent"[1]. The client and the user are indistinguishable for the token's lifetime.

This made sense for the world OAuth was built for. When you authorize a calendar app to access your Google Calendar, the app's requests are your requests because the app executes your explicit instructions.

But agents don't execute explicit instructions. They reason, they branch, and they make decisions based on context that didn't exist when the token was issued. "OAuth scopes are predefined and static; they cannot express lineage (which agent invoked whom) or context (why this action occurred)"[1].

Consider what happens in a multi-agent workflow:

Multi-agent authorization chain

1

Customer Request

Initial trigger

2

Intake Agent

authorized by: system

3

Refund Agent

Delegated from intake agent

4

Risk Agent

Delegated from refund agent

5

Treasury Agent

Delegated from risk agent

$50,000 transfer

Four agents, four delegation hops. OAuth wasn't designed for this. "A resource server at the end of the chain must be able to cryptographically verify the entire delegation path back to the original user, not just the final sub-agent making the request"[2].

The problem amplifies at scale: "An agent wields the delegated authority of a human but operates with the speed and scale of a machine"[2], creating a vastly amplified blast radius for a breach. We're not talking about a compromised user clicking a phishing link, but potentially thousands of autonomous decisions before anyone notices.

And OAuth breaks entirely across organizational boundaries.

OAuth assumes a single trusted authority, your IdP. But when your agent talks to a partner's agent, whose IdP validates the interaction? When your inventory agent negotiates with a customer's shopping agent, there's no shared auth server.

Traditional identity gives you authentication without accountability, authorization without delegation, and audit without proof.

Autonomous software needs all three, working together, across every boundary.

Identity Infrastructure for Reasoning Agents

What if identity worked the way autonomous software works?

Instead of tokens that represent "who holds this credential," imagine identity based on cryptographic key ownership. Instead of authorization requiring callbacks, imagine verification that happens locally. Instead of delegation meaning "forward the same token," imagine delegation meaning "issue a new, scoped credential."

The solution comes from identity standards designed for decentralized trust. Decentralized Identifiers (DIDs) let agents prove identity cryptographically[3], while Verifiable Credentials (VCs) encode scoped authority that anyone can verify offline[4].

The core ideas in plain terms

A Decentralized Identifier (DID) is a globally unique ID that you control. Unlike a username assigned by Google or an API key issued by your IdP, a DID is generated from a cryptographic key pair that belongs to the agent itself. No central registry. No dependency on someone else's server. The agent proves ownership by signing with its private key, and anyone can verify by checking the signature against the public key. DIDs became a W3C Recommendation in 2022 and are now a foundational web standard.

A Verifiable Credential (VC) is a digitally signed statement. Think of it as a tamper-proof certificate: "Agent X is authorized to perform refunds up to $5,000 until 3pm." The issuer signs it, the holder presents it, and any verifier can check the signature without calling back to the issuer. Credentials can encode any claim: permissions, roles, delegation chains, audit context. The W3C published Verifiable Credentials 2.0 as a full standard in 2025.

Why this matters for agents: Traditional identity requires a live connection to an authority. DIDs and VCs let agents carry proof with them, verify each other without network calls, and delegate authority that attenuates automatically. The trust model finally matches how autonomous software actually operates.

First Principles Deep Dive

Each agent gets a real identity.

Not a service account, not a forwarded token, but a cryptographic key pair that belongs to this specific agent, verifiable without calling anyone.

import os

from agentfield import Agent

app = Agent(

node_id="fraud-detection",

agentfield_server=os.getenv("AGENTFIELD_SERVER", "http://localhost:8080"),

# Automatically generates:

# - Agent DID: did:key:z6MkpTHR8VNs...

# - Every @app.reasoner() gets its own DID

# - Every execution produces a verifiable credential

Today, AgentField uses the did:key method, where the DID itself encodes the public key (Ed25519). Verification is instant and requires zero network lookups. The identity is the key. For organizations that need discoverable, web-hosted identities, AgentField also supports did:web.

Delegation becomes explicit.

When one agent delegates to another, it doesn't share a token. It issues a credential: "Agent B can perform refund transactions up to $5,000 for the next hour." The credential is signed by Agent A's private key, anyone can verify it, and no callbacks are required.

@app.reasoner()

async def check_fraud(application: dict) -> dict:

# Agent calls another agent - context propagates automatically

# The call carries verifiable proof of who authorized it

credit = await app.call(

"credit-bureau.check_score",

ssn=application["ssn"],

return {"credit": credit}

Each delegation step attenuates scope. The receiving agent gets a credential that's narrower than the issuing agent's authority. Permissions shrink as they propagate, by design.

Permissions travel with the request.

The credential carries everything needed to verify authority: the delegation chain, the constraints, the signatures. A verifier at the end of the chain can cryptographically trace authority back to the origin without trusting any intermediate system.

Audit becomes proof.

Every execution produces a credential. Not a log entry that someone promises is accurate, but a cryptographically signed record that can be verified offline, years later, without cooperation from any party.

The math works regardless of organizational boundary. Two agents from different companies establish trust by exchanging credentials and verifying signatures, with no shared IdP, no federation agreement, and no pre-arrangement required.

Not blockchain mysticism but straightforward public key cryptography applied to the specific problem of autonomous software.

DIDs give each agent a real, cryptographically verifiable identity. Verifiable Credentials encode delegation relationships with automatic scope attenuation. Together, they provide the accountability infrastructure that guided autonomy requires.

Trust Without Borders

Everything we've described so far works within your organization, but the real power emerges across boundaries.

Consider a customer's shopping agent negotiating with your inventory agent, which checks with a third-party logistics agent to guarantee same-day delivery. Three companies, three agent systems, one transaction.

Cross-organization transaction

Customer

Retailer

Logistics Co

Payment Co

Customer Request

Initial trigger

Customer's Shopping Agent

Retailer's Inventory Agent

Logistics Provider's Routing Agent

Payment Processor's Verification Agent

$2,000 purchase commitment

Four agents, four companies, four legal entities. No pre-existing trust relationship.

Traditional cross-company trust requires federation: shared identity providers, SAML agreements, mutual API key exchange. This worked when companies had a handful of integration partners, but it fails catastrophically when every company has agents that need to talk to every other company's agents.

You can't pre-federate with the entire economy.

But with DIDs and Verifiable Credentials, two agents from different organizations establish trust at dialogue initiation:

Exchange DIDs

Present relevant credentials

Verify signatures against public keys

No shared IdP, no federation setup, no pre-shared secrets.

The customer's agent has a credential from their identity provider, your inventory agent has a credential from your organization, and the logistics agent has a credential from the logistics company. Each verifies the others' authority without trusting each other's systems.

The agent economy becomes possible. Not agents replacing humans but agents transacting with other agents, across organizational boundaries, at machine speed, with cryptographic accountability.

When the customer claims they never authorized a purchase, you don't need to trust their logs or yours. You have the signed credential chain. The DID Resolution Bundle contains everything needed for a third party (a regulator, an arbitrator, a court) to verify the transaction years later, without cooperation from any party.

The receipt IS the proof.

# Export cross-organization transaction credentials

curl http://af-server/api/v1/did/workflow/wf-purchase-12345/vc-chain > audit.json

# Regulator verifies completely offline

# No AgentField server needed. No cooperation from other parties required.

af vc verify audit.json

"valid": true,

"type": "workflow",

"workflow_id": "wf-purchase-12345",

"signature_valid": true,

"format_valid": true,

"summary": {

"total_components": 4,

"valid_components": 4,

"total_dids": 4,

"resolved_dids": 4,

"total_signatures": 5,

"valid_signatures": 5

Guided autonomy requires accountable identity, accountable identity enables trust without borders, and trust without borders enables the agent economy.

What This Means for Your Team

How teams build and operate autonomous software changes.

For engineering teams:

Agents become first-class actors with real identity, partner integrations don't require federation setup, and multi-agent workflows are auditable by default. Not because you remembered to add logging, but because the credential chain is automatic.

No more "god-mode" API keys that grant sweeping access because it was easier than figuring out the right scopes. Permissions are explicit, delegatable, and auditable.

For security teams:

Every action has cryptographic attribution, with zero-trust across organizational boundaries. You can verify partner agents without trusting partner systems, and the blast radius is contained by design.

When something goes wrong, you know exactly which agent did what, under whose authority, at what time, even when that chain crosses organizational boundaries.

For compliance teams:

Audit trails that auditors can actually verify, offline, without cooperation from partner companies. The regulatory conversation shifts from "prove your logs are accurate" to "here's the cryptographic proof."

Regulated industries like finance, healthcare, and supply chain become accessible because the audit story finally makes sense.

For business development:

New partners onboard in minutes, not months. No "integration project" for agent-to-agent communication, just credential exchange.

The agent economy becomes possible.

The Trust Layer

In The AI Backend, we argued that your backend needs a reasoning layer. Software that can think.

Software that thinks needs to prove it. Not with logs someone promises are accurate, not with tokens that assume predictable behavior, and not with audit trails that require trusting someone's word, but with cryptographic identity infrastructure designed for autonomous software, infrastructure that enables delegation, local verification, and proof of every action.

We built AgentField with DIDs and Verifiable Credentials from day one. Not because we wanted to use fancy crypto, but because guided autonomy requires accountable identity.

The future of software is autonomous, and the future of identity is accountable. They're the same future.

The infrastructure for accountable autonomy is open source.

See the code | Start building

References

"Agentic JWT: A Secure Delegation Protocol for Autonomous AI Agents" (arXiv:2509.13597, Sept 2025)

"Identity Management for Agentic AI" (arXiv:2510.25819, Oct 2025)

"A Novel Zero-Trust Identity Framework for Agentic AI" (arXiv:2505.19301, May 2025)

"AI Agents with Decentralized Identifiers and Verifiable Credentials" (arXiv:2511.02841, Nov 2025)

Vyas et al., "Authenticated Delegation and Authorized AI Agents" (arXiv:2501.09674, Jan 2025)
