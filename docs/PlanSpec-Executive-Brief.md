# PlanSpec — Executive Brief

**An open standard for structured AI execution plans.**

---

## The Problem

When an AI agent plans work — writing code, modifying infrastructure, executing a multi-step task — that plan exists as a chat message. Unstructured. Unverifiable. Impossible to audit after the fact.

As AI agents take on higher-stakes work in enterprise environments, this becomes a critical gap:

- **No accountability** — there's no record of what the AI intended to do, what constraints it acknowledged, or what risks it identified
- **No validation** — there's no way to check if the plan is complete, consistent, or safe before execution begins
- **No oversight** — there's no boundary between what the AI is authorized to do autonomously and what requires human approval
- **No audit trail** — when something goes wrong, there's no artifact to trace the decision chain

## The Solution

**PlanSpec** defines a standard format for execution plans that are:

- **Human-readable** — a senior engineer or manager can review one without AI assistance
- **Machine-executable** — structured enough for an agent to follow step by step
- **Validatable** — constraints, success criteria, and scope boundaries are explicit and checkable
- **Auditable** — every decision has recorded rationale, every revision has history

## What's in a PlanSpec

A PlanSpec document captures six critical dimensions of any execution plan:

| Dimension | What It Answers |
|-----------|----------------|
| **Intent** | What are we doing and why? What does success look like? |
| **Scope** | What is explicitly in and out of bounds? What assumptions are we making? |
| **Approach** | What are the steps, their dependencies, and their complexity? |
| **Constraints** | What conditions must hold throughout execution? (Security, compliance, performance) |
| **Delegation** | What can the AI do autonomously vs. what needs human sign-off? |
| **Risks** | What could go wrong, how likely is it, and what's our mitigation? |

Every PlanSpec is versioned, revision-tracked, and carries a full decision history.

## Why an Open Standard?

Today, every AI planning tool invents its own format. Plans can't be shared between tools, reviewed by standardized processes, or audited against a common baseline.

An open standard means:

- **Tool interoperability** — any planning tool can produce PlanSpec documents; any execution tool can consume them
- **Independent validation** — a PlanSpec can be reviewed by a human, a tool, or a committee using consistent criteria
- **Organizational governance** — enterprises can adopt PlanSpec as a policy: "no AI agent executes without an approved PlanSpec"
- **Ecosystem growth** — validators, dashboards, audit tools, and comparison engines all build on the same format

## Real-World Example

A robotics company deploying autonomous systems in a refinery needs to ensure every AI-planned operation is reviewed before execution. A PlanSpec document for a refinery operation would specify:

- **Scope:** Which valves, sensors, and control systems are in play
- **Constraints:** Safety tolerances, regulatory compliance requirements, rollback procedures
- **Delegation:** Which steps the AI can execute autonomously vs. which require human approval
- **Success criteria:** Measurable conditions that confirm the operation completed safely
- **Risks:** What could fail, probability assessments, and contingency plans

The plan is reviewed by a safety engineer, approved, and handed to the execution system. If anything goes wrong, the PlanSpec is the artifact that investigators use to trace decisions.

## Current Status

- **Version 1.2.0** published and available under MIT license
- Full specification, JSON Schema, and examples at [github.com/Dazlarus-Industries/planspec](https://github.com/Dazlarus-Industries/planspec)
- Reference implementation (Planclave engine) in development by Dazlarus Industries
- Seeking early adopters and feedback from organizations deploying AI agents in production

## Who Should Care?

| Audience | Why |
|----------|-----|
| **Engineering leaders** | Standardize how AI agents plan and execute work across teams |
| **Security & compliance teams** | Gain auditability and oversight over AI-driven operations |
| **AI tool builders** | Adopt a proven format instead of inventing your own |
| **Enterprise architects** | Define governance policies for AI agent deployment |
| **Regulators & auditors** | Establish a baseline artifact for AI accountability |

## Contact

- **Repository:** [github.com/Dazlarus-Industries/planspec](https://github.com/Dazlarus-Industries/planspec)
- **Website:** [dazlarus.dev](https://dazlarus.dev)
- **Email:** darien@dazlarus.dev

---

*PlanSpec is developed by Dazlarus Industries and released under the MIT License.*
