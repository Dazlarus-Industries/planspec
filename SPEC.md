# PlanSpec — Schema Specification v1.2

**Status:** Draft for Public Comment  
**Version:** 1.2.0  
**Published:** 2026-06-26  
**License:** MIT  

---

## 1. What Is PlanSpec?

PlanSpec is an **open standard for structured AI execution plans**.

When an AI agent plans work — writing code, modifying infrastructure, executing a multi-step task — that plan should be more than an unstructured chat message. It should be a durable, versioned, machine-readable document with explicit scope, steps, constraints, success criteria, and human oversight boundaries.

A **PlanSpec document** is the artifact that captures all of this. It is:

- **Human-readable** — a senior developer can review and approve one without AI assistance
- **Machine-executable** — structured enough for an agent to follow step by step
- **Validatable** — constraints, success criteria, and scope boundaries are explicit and checkable
- **Auditable** — every decision has rationale, every revision has history

### Design Principles

1. **Execution semantics first** — Every field has behavioral meaning. `steps` have ordering and dependencies; `constraints` are validated; `success_criteria` are checked before completion.
2. **Human legibility is mandatory** — A PlanSpec that can only be interpreted by an LLM is a prompt, not a plan. Narrative rationale fields are required alongside structured fields.
3. **Stability over completeness** — A plan specifies enough to proceed confidently, not so much that it constrains legitimate implementation decisions.
4. **Revisability** — Plans are living artifacts. The schema supports revision history, change classification, and append-only revision chains.
5. **Specification ≠ Execution** — The PlanSpec captures intent. Execution traces are tracked separately. The plan is the contract; the execution record is the ledger.

### What PlanSpec Is Not

- A ticket or user story
- A chat response
- A code diff
- A project-management timeline
- A commit message

---

## 2. Top-Level Structure

A PlanSpec document contains these sections:

```
PlanSpec
├── identity              — Plan identification and revision tracking
├── intent                — What the plan is trying to achieve (immutable per session)
├── scope                 — Explicit inclusion/exclusion boundaries
├── approach              — Implementation strategy with ordered steps
│   └── steps[]           — Dependent, ranked execution steps
├── constraints           — Conditions that must hold throughout execution
├── success_criteria      — How to know the plan is complete
├── risks                 — Identified risks with mitigations
├── dependencies          — External prerequisites that must be available
├── delegation            — Human vs. agent authority boundaries
├── revision_history      — Append-only record of changes
└── metadata              — Schema version, timestamps, provenance
```

**Canonical serialization format:** JSON  
**Human display format:** Rendered Markdown (derived, never stored as the source of truth)

---

## 3. Field Definitions

### 3.1 `identity`

Establishes the plan's unique identity and revision position.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `plan_id` | string (UUID) | Yes | Globally unique identifier |
| `session_id` | string | Yes | The planning session that produced this plan |
| `revision` | integer ≥ 1 | Yes | Sequential revision number (1 = initial) |
| `parent_id` | string (UUID) \| null | Yes | `plan_id` of the previous revision. `null` for the initial revision. |
| `workspace_id` | string | No | Workspace or project context |
| `created_by` | string | Yes | User or agent identifier |
| `created_at` | string (ISO 8601) | Yes | Creation timestamp |

> **Note:** `revision` is a simple integer, not semver. The *schema version* (in `metadata.schema_version`) follows semver. See [§7](#7-versioning).

### 3.2 `intent`

The authoritative statement of what the plan is trying to accomplish. **Immutable across revisions** of the same session. If the intent changes fundamentally, start a new session.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `goal_statement` | string | Yes | Plain-language description of the primary goal |
| `motivation` | string | Yes | Why this goal matters in context |
| `success_definition` | string | Yes | One-sentence definition of success |
| `scope_context` | string | No | Brief description of the system/codebase context |

### 3.3 `scope`

Explicit inclusion and exclusion boundaries. The `out_of_scope` list is critical — plans without explicit exclusions tend to expand during execution.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `in_scope` | string[] | Yes | Explicitly included areas, components, or concerns (min: 1 item) |
| `out_of_scope` | string[] | Yes | Explicitly excluded areas with brief rationale |
| `assumptions` | string[] | No | Assumptions that must hold for the plan to be valid |
| `open_questions` | string[] | No | Unresolved questions needing answers before or during execution |

### 3.4 `approach`

The implementation strategy.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `overview` | string | Yes | Brief narrative of the overall approach |
| `rationale` | string | Yes | Why this approach was chosen |
| `alternatives_considered` | Alternative[] | No | Rejected alternatives with reasons |
| `steps` | Step[] | Yes | Ordered, dependent execution steps (min: 1 for non-trivial plans) |

**Alternative object:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `approach` | string | Yes | Description of the alternative |
| `rejection_reason` | string | Yes | Why it was rejected |

**Step object:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `step_id` | string | Yes | Unique step identifier within this plan |
| `title` | string | Yes | Short, action-oriented title |
| `description` | string | Yes | Detailed description of what needs to be done |
| `rationale` | string | No | Why this step is needed |
| `inputs` | string | No | What this step requires to proceed (free text) |
| `outputs` | string | No | What this step produces (free text) |
| `dependencies` | string[] | No | `step_id` values that must complete first |
| `parallelizable` | boolean | No | Whether this step can run in parallel with others |
| `estimated_complexity` | enum | No | One of: `xs`, `s`, `m`, `l`, `xl` |
| `delegation` | enum | No | One of: `human`, `agent`, `either`. Overrides `delegation.default_mode` for this step. |
| `acceptance_criteria` | string | Conditional | How to know this step is done. **Required** when `delegation` is `agent`. |

### 3.5 `constraints`

Conditions that must hold throughout execution.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `constraint_id` | string | Yes | Unique identifier |
| `type` | enum | Yes | One of: `technical`, `process`, `security`, `compliance`, `performance` |
| `description` | string | Yes | What the constraint requires |
| `validation_method` | string | Yes | How compliance can be checked |
| `blocking` | boolean | Yes | Whether violation halts execution |

### 3.6 `success_criteria`

Defines when the overall plan is complete. At least one criterion must have `weight: "primary"`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `criterion_id` | string | Yes | Unique identifier |
| `description` | string | Yes | What must be true for success |
| `verification` | enum | Yes | One of: `manual`, `automated`, `observable` |
| `weight` | enum | Yes | One of: `primary`, `secondary` |

### 3.7 `risks`

Identified risks with mitigations and contingencies.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `risk_id` | string | Yes | Unique identifier |
| `title` | string | Yes | Brief risk description |
| `probability` | enum | Yes | One of: `low`, `medium`, `high` |
| `impact` | enum | Yes | One of: `low`, `medium`, `high` |
| `description` | string | Yes | Full description |
| `mitigation` | string | Conditional | Mitigation strategy. **Required** when `probability` is `high`. |
| `contingency` | string | No | What to do if the risk materializes |
| `owner` | enum | No | One of: `human`, `agent`, `shared` |

### 3.8 `dependencies`

External prerequisites that must exist before execution begins.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `dependency_id` | string | Yes | Unique identifier |
| `description` | string | Yes | What the dependency is |
| `type` | string | No | Category (e.g., `service`, `library`, `access`, `data`) |
| `available` | boolean | Yes | Whether the dependency is currently available. Must be `true` for execution readiness. |

### 3.9 `delegation`

Declares what is authorized for autonomous execution vs. human-gated approval. This is the **human oversight contract**.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `default_mode` | enum | Yes | One of: `human_approval`, `agent_autonomous`, `hybrid` |
| `step_overrides` | StepOverride[] | No | Per-step delegation overrides |
| `escalation_policy` | EscalationPolicy | No | What to do when things go sideways |

**StepOverride:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `step_id` | string | Yes | Step identifier (must reference a valid step) |
| `mode` | enum | Yes | One of: `human_approval`, `agent_autonomous` |
| `rationale` | string | No | Why this step requires the specified mode |

**EscalationPolicy:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `on_ambiguity` | enum | No | One of: `pause`, `fail`, `proceed_with_best_guess` |
| `on_constraint_violation` | enum | No | One of: `pause`, `fail`, `notify` |
| `on_unexpected_scope` | enum | No | One of: `pause`, `fail`, `notify` |

### 3.10 `revision_history`

Append-only record of revisions. Each entry captures the change metadata for that revision.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `revision` | integer ≥ 1 | Yes | Revision number |
| `change_type` | enum | Yes | One of: `major`, `minor`, `patch` |
| `change_summary` | string | Yes | Human-readable description of what changed |
| `trigger` | enum | No | One of: `user_feedback`, `agent_suggestion`, `clarification`, `error_correction` |
| `changes` | ChangeDiff[] | No | Structured diffs |
| `created_at` | string (ISO 8601) | Yes | When this revision was created |

**ChangeDiff:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | enum | Yes | One of: `addition`, `removal`, `modification`, `reorder` |
| `section` | string | Yes | Which section changed (e.g., `steps`, `scope`, `constraints`) |
| `field` | string | No | Specific field changed |
| `summary` | string | Yes | Human-readable description of the change |

**Change type semantics:**

| Type | Description |
|------|-------------|
| `major` | Approach overhaul — complete replan within same intent and scope |
| `minor` | New steps, step reordering, or material approach change |
| `patch` | Clarification, typo fix, or non-structural metadata update |

> **Note:** Intent and scope are immutable within a session. A change to intent or scope requires creating a new session, not a revision. `major` revisions represent approach overhauls only.

### 3.11 `metadata`

Schema and provenance metadata.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema_version` | string (semver) | Yes | PlanSpec schema version this document conforms to. Current: `1.2.0` |
| `tool` | string | No | Name of the tool that generated this plan |
| `tool_version` | string | No | Version of the generating tool |
| `tags` | string[] | No | Free-form tags for categorization |
| `language` | string | No | Primary language of the plan content (BCP 47 code, e.g., `en`) |

---

## 4. Revision Semantics

### 4.1 Revision Chain

Revisions form a linear, append-only chain via `identity.parent_id`:

```
revision 1  (initial; change_type: major; parent_id: null)
    │
    └── revision 2  (approach revised; change_type: minor; parent_id: <rev1 plan_id>)
            │
            └── revision 3  (step added; change_type: patch; parent_id: <rev2 plan_id>)
                    │
                    └── revision 4  (approach overhauled; change_type: major; parent_id: <rev3 plan_id>)
```

Rules:
- Revisions are **append-only**. No revision is ever deleted.
- The **current revision** is the latest in the chain.
- Every revision has a corresponding file artifact (e.g., `plan-v1.json`, `plan-v2.json`).

### 4.2 Revision Authority

| Revision Type | Who Can Initiate | Who Must Approve |
|---------------|-----------------|-----------------|
| Patch | Agent or Human | Human (or auto-approved by tooling) |
| Minor | Agent or Human | Human |
| Major | Agent or Human | Human |

### 4.3 Intent Immutability

`intent` is immutable across revisions of the same session. If the goal changes fundamentally, create a new session. Major revisions can overhaul the *approach* but not the *intent*.

---

## 5. Validation Rules

Validation operates at three levels. A plan must pass Level 1 to be structurally valid, Level 2 to be semantically consistent, and Level 3 to be ready for execution.

### Level 1 — Schema Validation

- All required fields are present
- All IDs are non-empty strings
- `identity.revision` is a positive integer
- `identity.created_at` is valid ISO 8601
- `metadata.schema_version` is a valid semver string
- `approach.steps[].dependencies` reference valid `step_id` values within the same plan
- `delegation.step_overrides[].step_id` references valid step IDs
- No circular dependencies in `steps[].dependencies`

### Level 2 — Semantic Validation

- `intent.goal_statement` is not empty
- `scope.in_scope` has at least one item
- `success_criteria` has at least one entry with `weight: "primary"`
- `approach.steps` has at least one item for non-trivial plans
- Each step's `delegation` value is consistent with `delegation.default_mode` or has an explicit override
- Steps with `delegation: "agent"` have explicit `acceptance_criteria`

### Level 3 — Execution Readiness Validation

- All `scope.open_questions` are resolved (empty or answered)
- All `risks` with `probability: "high"` have documented `mitigation`
- All `dependencies` entries have `available: true`
- `delegation.default_mode` is explicitly set

---

## 6. Extensibility

### 6.1 Domain Extensions

The base schema is domain-agnostic. Domain specializations extend it via namespaced `x-*` fields without modifying the core:

```json
{
  "metadata": {
    "schema_version": "1.2.0"
  },
  "x-codespec": {
    "codebase_context": {
      "primary_language": "typescript",
      "framework": "fastify",
      "test_framework": "vitest"
    }
  }
}
```

Conventions:
- Extension keys use the `x-<domain>:` prefix pattern
- The base schema **ignores unknown `x-*` fields** for forward compatibility
- Extensions must not override or redefine base field semantics

### 6.2 Custom Constraint Types

Domain implementations can register custom constraint `type` values. Unknown constraint types produce **warnings**, not errors, to allow forward compatibility.

---

## 7. Versioning

### 7.1 Schema Version vs. Plan Revision

These are distinct:
- **Schema version** (`metadata.schema_version`): the version of the PlanSpec format itself
- **Plan revision** (`identity.revision`): the revision number of a specific plan document

### 7.2 Schema Versioning Policy

Schema versions follow [semver](https://semver.org/):
- **Patch** (1.2.x): backward-compatible additions of optional fields
- **Minor** (1.x.0): backward-compatible additions of new sections or field types
- **Major** (x.0.0): breaking changes to existing fields or semantics

### 7.3 Cross-Version Compatibility

Consumers should handle older schema versions gracefully:
- If `schema_version` is missing: assume `1.0.0`
- If major version exceeds consumer's: display warning; attempt best-effort parsing
- If minor version exceeds consumer's but major matches: render known fields; ignore unknown fields

---

## 8. Serialization

### 8.1 Canonical Format: JSON

JSON is the canonical serialization format. All APIs, artifact files, and interchange use JSON.

### 8.2 Display Format: Rendered Markdown

For human review, PlanSpecs may be rendered to structured Markdown. This is a **view transform**, not a storage format. Plans are never stored as Markdown.

### 8.3 Future: YAML

A YAML representation is planned for human editing. It will be an input format only; the canonical stored format remains JSON.

---

## 9. Machine Readability Requirements

A conforming PlanSpec consumer must support:

1. **Dependency graph extraction** from `steps[].dependencies`
2. **Delegation mode lookup** per step
3. **Success criteria** evaluation status tracking
4. **Constraint compliance** checking
5. **Revision history** traversal via `identity.parent_id`
6. **Scope boundary enforcement** (detect when a proposed action falls outside `scope.in_scope`)

---

## 10. Quickstart

### Minimal Valid PlanSpec

The smallest structurally valid PlanSpec with all required fields:

```json
{
  "identity": {
    "plan_id": "00000000-0000-0000-0000-000000000001",
    "session_id": "sess-001",
    "revision": 1,
    "parent_id": null,
    "created_by": "human",
    "created_at": "2026-06-26T12:00:00Z"
  },
  "intent": {
    "goal_statement": "Update the README with installation instructions",
    "motivation": "New users cannot figure out how to install the package",
    "success_definition": "README contains a working installation section"
  },
  "scope": {
    "in_scope": ["README.md installation section"],
    "out_of_scope": ["Rewriting the entire README", "Adding new documentation pages"]
  },
  "approach": {
    "overview": "Add a standard installation section to the README after the project description.",
    "rationale": "Most package registries expect standard npm/pip install instructions near the top.",
    "steps": [
      {
        "step_id": "s1",
        "title": "Write installation section",
        "description": "Add an 'Installation' heading with npm install instructions.",
        "dependencies": [],
        "acceptance_criteria": "Section is present and instructions work when followed"
      }
    ]
  },
  "success_criteria": [
    {
      "criterion_id": "sc1",
      "description": "Following the README instructions installs the package successfully",
      "verification": "manual",
      "weight": "primary"
    }
  ],
  "delegation": {
    "default_mode": "agent_autonomous"
  },
  "metadata": {
    "schema_version": "1.2.0"
  }
}
```

### Validation Checklist

Before publishing or executing a PlanSpec, verify:

- [ ] `identity.plan_id` is a valid UUID
- [ ] `identity.revision` is a positive integer
- [ ] `identity.parent_id` is `null` (for revision 1) or references a valid prior `plan_id`
- [ ] `intent.goal_statement` is non-empty
- [ ] `scope.in_scope` has ≥ 1 entry
- [ ] `scope.out_of_scope` has ≥ 1 entry
- [ ] `approach.steps` has ≥ 1 step
- [ ] All `steps[].dependencies` reference valid `step_id` values
- [ ] No circular dependencies exist in the step graph
- [ ] `success_criteria` has ≥ 1 entry with `weight: "primary"`
- [ ] All steps with `delegation: "agent"` have `acceptance_criteria`
- [ ] `metadata.schema_version` is `"1.2.0"` (or compatible)
- [ ] `delegation.default_mode` is explicitly set (not omitted)

---

## Appendix A: Open Questions

These are unresolved design questions that may be addressed in future schema versions:

1. **Nested plans** — Should PlanSpec support hierarchical child plans, or is flat single-level sufficient?
2. **Plan templates** — Should the schema support declaring a plan as derived from a template?
3. **Step artifacts** — Should steps have structured output declarations (file paths, test results)?
4. **Multi-agent steps** — How should delegation work when a step requires sequential involvement from multiple agents?
5. **Plan merging** — What is the reconciliation strategy when revisions diverge across branches?

---

## Appendix B: Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.2.0 | 2026-06-26 | Formalized field definitions with types; resolved contradictions (revision as integer vs semver); defined `dependencies`, `revision_history`, and `metadata` sections; added structured change diffs; removed product-phase references; added examples and validation checklist |
| 1.1.0 | 2026-05-17 | Pre-formalization architecture document (internal) |
| 1.0.0 | — | Initial conceptual model (internal) |

---

*PlanSpec is an open standard developed by [Dazlarus Industries](https://github.com/Dazlarus-Industries). The schema is released under the [MIT License](LICENSE).*
