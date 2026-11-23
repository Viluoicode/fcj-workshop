---
title: "AWS for Software and Technology Companies Day"
date: 2026-10-08
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---

# AWS for Software and Technology Companies Day

## Event Information

- **Date:** 08 October 2026
- **Location:** Sheraton Saigon Grand Opera
- **Organizer:** Amazon Web Services (AWS)
- **Workshop track attended:** Track B - Build & Deploy AI Agents on Amazon Bedrock AgentCore Using Vibe Coding with Kiro

## Event Overview

The event brought together AWS specialists and technology practitioners to discuss production-ready AI agents, high-quality data products for AI, agentic cloud migration, and practical development with Kiro and Amazon Bedrock AgentCore. The program included three technical presentations, two parallel workshop tracks, a customer spotlight, and closing remarks.

## Welcome and Opening

**Speaker:** Hung Gia - Manager, Solutions Architecture, AWS

The opening introduced the event agenda and its focus on agentic AI, modern data foundations, cloud migration, modernization, and hands-on learning.

## Session 1 - From Demo to Deployment: Solving Agentic AI's Toughest Challenges

**Speaker:** Dzung Ha - Solutions Architect, AWS

The session explained the gap between an AI proof of concept and a production-ready system through four pillars:

- **Operational Excellence:** observability, cost control, scale, latency, and governance.
- **Data and Context:** data quality, retrieval, RAG, grounding, and short-term and long-term memory.
- **Reliability:** deterministic code, evaluations, continuous testing, monitoring, and recovery.
- **Trust:** identity, session isolation, policies, security boundaries, privacy, and guardrails.

The session also introduced Amazon Bedrock AgentCore Memory, AgentCore Evaluations, and Amazon Bedrock Guardrails. The main lesson was that a working model alone is not enough; a production agent also requires reliable data, monitoring, evaluation, security, and governance.

What I found especially useful was the speaker's progression from a familiar problem - many AI proofs of concept never reach production - to a structured production framework. Instead of presenting individual AWS services in isolation, the speaker connected each service to an operational question: whether the system can be observed, whether its context is trustworthy, whether its behavior can be evaluated, and whether it remains within defined boundaries. The examples around context windows, memory, RAG quality, model evaluation, and guardrails helped me see an AI agent as a complete software system rather than only a model and prompt.

I also learned an important engineering principle from this session: deterministic code should still be used for calculations, validation, and rule-based work, while the agent should be reserved for tasks that genuinely require reasoning. This approach can reduce cost and latency while making the overall system easier to test and troubleshoot.

## Session 2 - Building High-Quality Data Products for AI Agents

**Speaker:** Binh Tran - Senior Solutions Architect, AWS

This session described five characteristics of AI-ready data: trusted, contextual, traceable, governed, and operational. The main topics included:

- Data product ownership, operating models, and service-level expectations.
- Data contracts covering schemas, quality rules, freshness, versioning, and access.
- Enforcing data quality close to the source with services such as AWS Glue Data Quality.
- Technical and business metadata, lineage, semantic layers, business glossaries, and ontologies.
- Fine-grained access control with IAM, AWS Lake Formation, Cedar policies, and agent identity.
- Interoperable access through open table formats and AWS MCP Servers.
- RAG, vector and graph retrieval, Text-to-SQL, and safeguards before an agent changes a live system.

The key takeaway was that agents need explicit business meaning, validated data, traceability, freshness guarantees, and restricted access before they can make reliable decisions or take actions.

The presentation style in this session was valuable because the speaker used one data error and followed it through the source, pipeline, catalog, semantic layer, agent, and final action. This made the consequences of weak data quality easy to understand: an error may not cause a technical failure, but it can still lead an agent to make a confident and harmful decision. The data-contract example clarified how ownership, schema, business meaning, quality rules, freshness, versioning, and access should be defined together instead of being handled as separate concerns.

Several concepts were new or became clearer to me during this session. I learned why technical metadata alone is insufficient for an agent and how a business glossary, semantic layer, ground-truth examples, and entity relationships provide the context that people often understand implicitly. I also gained a clearer view of layered authorization: an agent needs its own identity, a limited service role, permissions delegated on behalf of a user, filtered access to data, and policy checks before calling tools or APIs. This changed my understanding of data preparation from simple cleaning and transformation into the design of a governed product that an agent can safely consume.

## Session 3 - Migration at Scale with Agentic AI: AWS Transform and Beyond

**Speaker:** Taiki Dang - Solutions Architect, Migration & Modernize | Security, AWS

The session covered the evolution from traditional migration to AI-assisted and agentic migration. It introduced AWS Transform and its integration with Kiro, Amazon Bedrock AgentCore, Cloud Migration Factory, and AWS Transform for VMware.

Three migration patterns were presented:

- **Assist:** use AI on top of a proven Cloud Migration Factory execution model.
- **Automate:** use AWS Transform across the migration lifecycle with human approval gates.
- **Orchestrate:** combine AWS Transform, Kiro, and AgentCore agents for migration and coordination work.

The session also discussed rehosting, replatforming, containerization, Windows and mainframe modernization, custom transformations, and customer examples from global organizations and Vietnam. The main lesson was that agents can automate migration workstreams while humans retain responsibility for important decisions and approvals.

The speaker made a complex migration topic easier to follow by first explaining the limitations of traditional, human-coordinated migration and then comparing it with AI-assisted and agentic approaches. The three patterns - Assist, Automate, and Orchestrate - gave me a practical way to understand that organizations do not need to adopt the same level of automation. The appropriate approach depends on compliance requirements, existing tools, team readiness, and the need for human approval, not only on the number of servers or applications.

I learned that agentic migration is not simply about generating scripts faster. It also involves discovery, planning, landing-zone and network preparation, worklog traceability, approval gates, execution, validation, and coordination across teams. The examples of AWS Transform as the execution engine, Kiro as the conversational interface, and AgentCore agents handling coordination tasks showed how multiple tools can share context while people continue to control high-impact decisions.

## Learning from the Speakers and Session Structure

One of my main reasons for attending this event was to observe how AWS specialists communicate advanced technical topics. The speakers did not begin with long lists of services. They first established a real problem, explained why existing approaches were insufficient, and then introduced architecture patterns and AWS capabilities as responses to that problem. This made each session easier to follow and gave the technical details a clear purpose.

I noticed several presentation techniques that I can apply to my own technical work:

- Start with the business or operational risk before describing the technology.
- Organize a large topic into a small number of memorable pillars or patterns.
- Use one scenario throughout a session so the audience can follow cause and effect.
- Compare alternatives and trade-offs instead of presenting one service as the answer to every problem.
- Connect architecture diagrams to responsibilities, permissions, data flow, and failure handling.
- Finish with concrete takeaways so the audience understands what should change in practice.

The three sessions also connected naturally. Session 1 described the requirements for a production-ready agent. Session 2 explained the trusted and governed data foundation that such an agent needs. Session 3 expanded the same ideas into a large operational use case in which agents, tools, policies, approval gates, and human teams work together. This progression helped me build a more complete understanding of Agentic AI at both application and organizational levels.

## Workshop Briefing

The event offered two parallel workshop tracks:

- **Track A:** Build with Kiro: From Spec to Working Application.
- **Track B:** Build & Deploy AI Agents on Amazon Bedrock AgentCore Using Vibe Coding with Kiro.

I selected and participated in **Track B**.

## Track B - Build & Deploy AI Agents on Amazon Bedrock AgentCore Using Vibe Coding with Kiro

**Level:** Intermediate to Advanced

### Workshop Goals

- Build an AI agent using Kiro.
- Develop and extend the agent incrementally through guided steps.
- Prepare and deploy the agent to Amazon Bedrock AgentCore.
- Explore the requirements for moving an AI agent from a prototype toward a cloud-based, production-oriented environment.

### My Progress and Result

I completed approximately half of the guided Track B workshop. I practiced the initial stages of building an AI agent with Kiro and explored the workflow for developing the agent step by step and preparing it for deployment on Amazon Bedrock AgentCore.

Although I did not complete the full workshop journey, the hands-on session gave me practical exposure to agent development and production-oriented deployment concepts. This result is recorded as partial workshop progress and does not claim that the final deployment was completed.

### Hands-on Learning Experience

The workshop helped convert the morning's concepts into a development workflow. Working with Kiro showed me how an AI-assisted environment can guide implementation through context, instructions, and incremental tasks instead of producing an entire solution in one uncontrolled step. I practiced following the workshop sequence, reviewing the generated work, and understanding how the local agent-development process was intended to connect to Amazon Bedrock AgentCore.

Completing only about half of the track was still useful because it revealed the difference between understanding an architecture slide and carrying out the implementation. Each stage depends on correct project context, environment preparation, configuration, and verification. The unfinished portion also gave me a clearer learning target: revisit the remaining steps, complete the AgentCore deployment flow, and verify the agent after deployment rather than treating deployment itself as the final result.

## Overall Learning Experience

This event was primarily a learning opportunity for me. Beyond the individual AWS services, I gained a broader way of thinking about production systems: start from a real problem, define data and security boundaries, make behavior observable and testable, keep humans involved in high-impact decisions, and choose automation according to the operating environment.

The event also helped me identify subjects for further study, including AgentCore runtime components, memory and evaluation, data contracts, metadata and semantic layers, Cedar-based authorization, MCP integration, and the use of AWS Transform for migration and modernization. Observing how the speakers connected these topics to practical scenarios gave me a useful reference for both future learning and future technical presentations.

## Key Takeaways

- Production AI agents require observability, evaluation, reliable context, security, and governance in addition to model capabilities.
- AI-ready data must be validated, fresh, traceable, explicitly defined, and protected by appropriate access controls.
- Agentic AI can support cloud migration and modernization while preserving human approval for important decisions.
- Kiro and Amazon Bedrock AgentCore provide a practical path for building and preparing AI agents for cloud deployment.
- Completing part of Track B helped me understand the initial agent-development workflow and identify the remaining steps required for a complete deployment.

## Event Photos

![Check-in photo in front of the event backdrop, Thursday 8 October 2026, Ho Chi Minh City](../../Image/event%202/checkin.jpg)

*Check-in at the event backdrop on 8 October 2026.*

![Group photo with the AWS gifts received at the event](../../Image/event%202/anh1.jpg)

*With fellow interns after the sessions, holding the AWS gifts.*

![Workshop participants holding AWS gift boxes in front of the event backdrop](../../Image/event%202/anh2.jpg)

*Group photo of participants at the end of the day.*
