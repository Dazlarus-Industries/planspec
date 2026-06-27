# PlanSpec

**An open standard for structured AI execution plans.**

[![Spec Version](https://img.shields.io/badge/spec-v1.2.0-blue)](SPEC.md)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

---

When an AI agent plans to do something — write code, modify infrastructure, execute a multi-step task — the plan should be more than a chat message. It should be a durable, versioned, machine-readable document with explicit scope, steps, constraints, success criteria, and human oversight boundaries.

That's what a PlanSpec is.

## Why PlanSpec?

AI agents are increasingly tasked with real work: writing code, managing infrastructure, executing complex workflows. Today, the "plan" is usually just an LLM response in a chat — unstructured, unverifiable, and impossible to audit after the fact.

PlanSpec fixes this by defining a standard format for execution plans that:

- **Humans can read** — a senior developer should be able to review and approve a PlanSpec without AI assistance
- **Machines can execute** — structured enough for an agent to follow step by step
- **Reviewers can validate** — constraints, success criteria, and scope boundaries are explicit
- **Auditors can trace** — every decision has rationale, every revision has history

## Quick Example

```json
{
  "identity": {
    "plan_id": "550e8400-e29b-41d4-a716-446655440000",
    "revision": 1,
    "parent_id": null,
    "created_at": "2026-06-26T10:00:00Z"
  },
  "intent": {
    "goal_statement": "Add password reset flow to the auth service",
    "success_definition": "Users can request and complete a password reset via email"
  },
  "scope": {
    "in_scope": ["POST /auth/reset-request", "POST /auth/reset-confirm", "Email template"],
    "out_of_scope": ["Social login changes", "Session management refactoring"]
  },
  "approach": {
    "steps": [
      {
        "step_id": "s1",
        "title": "Create reset token model",
        "delegation": "agent",
        "dependencies": []
      },
      {
        "step_id": "s2",
        "title": "Implement reset-request endpoint",
        "delegation": "agent",
        "dependencies": ["s1"]
      }
    ]
  },
  "success_criteria": [
    {
      "description": "Reset flow works end-to-end in staging",
      "verification": "automated",
      "weight": "primary"
    }
  ],
  "delegation": { "default_mode": "agent_autonomous" },
  "metadata": { "schema_version": "1.2.0" }
}
```

See the [**full examples**](examples/) for plans at different complexity levels:

| Example | Complexity | Description |
|---------|-----------|-------------|
| [Documentation update](examples/01-minimal-documentation-update.json) | Minimal | Single-file README change — the smallest valid PlanSpec |
| [Password reset flow](examples/02-moderate-password-reset.json) | Moderate | Multi-step backend feature with security constraints, risks, and delegation overrides |
| [Serverless migration](examples/03-complex-serverless-migration.json) | Complex | 8-step infrastructure migration with revision history, weighted traffic shifting, and compliance constraints |

## What's in the Spec

| Section | Purpose |
|---------|---------|
| **identity** | Unique ID, revision chain, provenance |
| **intent** | What the plan is trying to achieve (immutable across revisions) |
| **scope** | Explicit in-scope and out-of-scope boundaries |
| **approach** | Implementation strategy with ordered, dependent steps |
| **constraints** | Conditions that must hold throughout execution |
| **success_criteria** | How to know when the plan is complete |
| **risks** | Identified risks with mitigations |
| **dependencies** | External prerequisites that must be available |
| **delegation** | What's authorized for autonomous execution vs. human approval |
| **revision_history** | Append-only record of changes with structured diffs |
| **metadata** | Schema version, tool provenance, tags |

**Full specification:** [SPEC.md](SPEC.md)

## Schema Version

Current: **v1.2.0**

The schema follows [semver](https://semver.org/). Breaking changes increment the major version; backward-compatible additions increment minor or patch versions.

Domain extensions (CodeSpec, ProductSpec, etc.) extend the base schema via namespaced `x-*` fields without modifying the core.

## Quickstart

1. **Read the spec:** [SPEC.md](SPEC.md)
2. **Copy an example** from [`examples/`](examples/) as a starting template
3. **Validate your plan** using the [validation checklist](SPEC.md#10-quickstart) (Level 1–3 rules)
4. **Optionally:** build a validator using the typed field definitions in [SPEC.md §3](SPEC.md#3-field-definitions)

## Implementations

- **[Planclave](https://github.com/Dazlarus-Industries)** — Engine that generates PlanSpec documents through a multi-agent council review process. *In development by Dazlarus Industries.*

Want to build a PlanSpec-compatible tool? The spec is open and MIT-licensed. Fork the examples, open an issue, or reach out.

## Design Principles

1. **Execution semantics first** — every field has behavioral meaning
2. **Human legibility is mandatory** — a plan you need AI to interpret is a prompt, not a plan
3. **Stability over completeness** — specify enough to proceed, not so much it constrains
4. **Revisability** — plans are living artifacts with full revision history
5. **Specification ≠ execution** — the plan is the contract; the execution record is separate

## License

The PlanSpec schema and specification documents are released under the **[MIT License](LICENSE)**.

The goal is maximum adoption: anyone should be able to implement, extend, and build tools around PlanSpec without restriction.

## Community

- 🌐 [dazlarus.dev](https://dazlarus.dev)
- 📝 [Blog](https://dazlarus.dev/blog)
- 🐙 [Dazlarus Industries on GitHub](https://github.com/Dazlarus-Industries)

---

*PlanSpec is developed by [Dazlarus Industries](https://github.com/Dazlarus-Industries).*
