# PlanSpec — One-Pager

## What

An **open standard (MIT)** for structured AI execution plans. When an AI agent needs to do real work — write code, modify infrastructure, operate systems — PlanSpec defines the format for the plan that gets reviewed, approved, and executed.

## Why It Matters

AI agents are moving from chat to execution. Today, the "plan" is an unstructured LLM response — no accountability, no validation, no audit trail. PlanSpec fixes that.

## How It Works

A PlanSpec document captures:

- **What** the AI will do (scoped steps with dependencies)
- **Why** this approach (recorded rationale and alternatives considered)
- **What's off-limits** (explicit scope exclusions)
- **What could go wrong** (risks with mitigations)
- **What requires human approval** (delegation boundaries)
- **How we know it's done** (measurable success criteria)

Every plan is versioned, revision-tracked, and machine-validatable.

## The Standard

| | |
|---|---|
| **Version** | 1.2.0 |
| **License** | MIT |
| **Format** | JSON (canonical), Markdown (rendered view) |
| **Schema** | JSON Schema Draft 07 |
| **Repository** | [github.com/Dazlarus-Industries/planspec](https://github.com/Dazlarus-Industries/planspec) |

## Who It's For

- **Engineering teams** deploying AI agents for code, infrastructure, or operations
- **Security/compliance** teams who need oversight and audit trails for AI-driven work
- **Tool builders** who want a proven plan format instead of inventing their own
- **Enterprise architects** defining governance policies for AI agent deployment

## Get Involved

- Read the spec: [SPEC.md](https://github.com/Dazlarus-Industries/planspec/blob/main/SPEC.md)
- Try the examples: [examples/](https://github.com/Dazlarus-Industries/planspec/tree/main/examples)
- Open an issue with feedback or questions
- Reach out: darien@dazlarus.dev

---

*PlanSpec is an open standard developed by Dazlarus Industries.*
