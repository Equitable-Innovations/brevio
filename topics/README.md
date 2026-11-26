# Topics: Product Ideation & Architectural Decision Framework

This is the `topics/` workspace for **`brevio` (Token Optimized Software Harness)**.

In the initial product ideation stage, and for the full life of the project, the team uses this directory to work
together. The team brainstorms, evaluates architecture, analyzes trade-offs, and builds agreement here.

When the team agrees on a topic, the conclusions of the topic graduate into authoritative specifications. These
specifications are in the [`build-docs/`](../build-docs/README.md) directory.

---

## Table of Contents

1. [Role & Relationship: Topics vs. Build-Docs](#role--relationship-topics-vs-build-docs)
2. [Topic Lifecycle & State Machine](#topic-lifecycle--state-machine)
3. [File Naming & Organization Conventions](#file-naming--organization-conventions)
4. [Structured Topic Template](#structured-topic-template)
5. [Conversion Pipeline: From Topic to Build-Docs](#conversion-pipeline-from-topic-to-build-docs)
6. [Build-Docs Domain Mapping Guide](#build-docs-domain-mapping-guide)
7. [Topic Registry](topic-registry.md)

---

## Role & Relationship: Topics vs. Build-Docs

Keep a clear boundary between `topics/` and [`build-docs/`](../build-docs/README.md). This keeps the documentation
clean:

| Dimension            | `topics/` (Ideation & Deliberation)                                                                                 | `build-docs/` (Canonical Specification)                                                  |
|:---------------------|:--------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------|
| **Purpose**          | Explore ideas, debate alternatives, evaluate token economics, record the agreement.                                 | The authoritative blueprint and specification for the build of `brevio`.                 |
| **State**            | Divergent, exploratory, collaborative.                                                                              | Convergent, normative, prescriptive.                                                     |
| **History & Debate** | Keeps the discussions, the rejected alternatives, the trade-offs, and the reasons for each decision.                | Contains no past decisions or alternatives. Shows only the current-state design.         |
| **Lifecycle**        | Moves through states (`PROPOSED` &rarr; `GRADUATED` / `REJECTED`). After graduation, it is an immutable ADR record. | Living documents that the team maintains during software development. The team changes them only through revisions. |
| **Primary Audience** | Product designers, architects, and engineers during ideation.                                                       | Implementers, agentic LLMs that execute tasks, automated code generators.                |

> [!IMPORTANT]
> [`build-docs/README.md`](../build-docs/README.md) sets this rule. All documents in `build-docs/` must show the
current state of the design. They must contain no references to past decisions. The debate and history stay
permanently in the `topics/` directory.

---

## Topic Lifecycle & State Machine

Each topic moves through a structured sequence, from its start to graduation:

```mermaid
flowchart LR
    A([Proposed&nbsp;💡]) --> B([In Discussion&nbsp;💬])
B --> C([Decided&nbsp;⚖])
B --> D([Deferred&nbsp;⏸])
B --> E([Rejected&nbsp;🚫])
C --> F([Graduated to \n Build-Docs&nbsp;🎓])
```

### State Definitions

1. **`PROPOSED`**: The topic has a problem statement, context, and initial motivation. The team can do an initial
   review.
2. **`IN_DISCUSSION`**: The discussion is active. The team compares options, technical spikes, token-efficiency models,
   and architectural effects.
3. **`DECIDED`**: The team agreed, or made an architectural decision. The topic records the chosen solution and the
   reasons. All open questions are resolved.
4. **`GRADUATED`**: The decision is copied into canonical specifications in [
   `build-docs/`](../build-docs/README.md). The topic document gets cross-references and is then locked.
5. **`DEFERRED`**: The topic is valid, but the team moved it to a later milestone or to a post-MVP phase.
6. **`REJECTED`**: The team explored the topic and rejected it. The topic keeps the reasons for future reference, so
   that the team does not debate a settled decision again.

---

## File Naming & Organization Conventions

- File path: `topics/TOPIC-###_<kebab-case-slug>.md`
- Topic numbering: Three-digit sequential numbers, padded with zeros (`001`, `002`, `003`, ...).
- Template: Copy from [`template.md`](template.md).

### Examples:

- `topics/TOPIC-001_cli-command-tree-and-dispatch.md`
- `topics/TOPIC-002_xhtml-schema-tag-structure.md`
- `topics/TOPIC-003_mcp-vs-rest-service-layering.md`
- `topics/TOPIC-004_static-ui-viewer-technology.md`

---

## Structured Topic Template

Each topic document must use this standard format. The format makes sure that the analysis is complete and that the
conversion into `build-docs/` is fast:

```markdown
# TOPIC-###: <Topic Title>

| Metadata               | Value                                                                                           |
|:-----------------------|:------------------------------------------------------------------------------------------------|
| **Topic ID**           | `TOPIC-###`                                                                                     |
| **Status**             | `PROPOSED` \| `IN_DISCUSSION` \| `DECIDED` \| `GRADUATED` \| `DEFERRED` \| `REJECTED`           |
| **Facilitator**        | @username                                                                                       |
| **Participants**       | @username1, @username2                                                                          |
| **Created Date**       | YYYY-MM-DD                                                                                      |
| **Last Updated**       | YYYY-MM-DD                                                                                      |
| **Target Build-Docs**  | `build-docs/...`                                                                                |
| **Impacted Domains**   | `CLI` \| `MCP` \| `Static UI` \| `Work Items` \| `Wiki` \| `OpenAPI` |

---

## 1. Problem Statement & Context

- **Problem**: What problem are we solving? What pain point or capability does this address?
- **Context**: Why are we addressing this during the product ideation stage? What triggered this topic?

## 2. Goals & Non-Goals

- **Goals**: What must this topic accomplish?
- **Non-Goals**: What is deliberately out of scope for this topic?

## 3. Token Optimization & Performance Impact

*Because `brevio` is built for token conservation and speed, how does this topic align with those core tenets?*

- Estimated impact on LLM token consumption (prompt size, response overhead, context window usage).
- Speed / local processing benefits (e.g., local XHTML parsing vs. remote cloud calls).

## 4. Proposed Options & Trade-Off Analysis

### Option A: <Name>

- **Description**: Summary of the approach.
- **Pros**:
    - Point 1
    - Point 2
- **Cons**:
    - Point 1
    - Point 2
- **Token / Latency Cost**: Evaluation of overhead.

### Option B: <Name>

- **Description**: Summary of the approach.
- **Pros**: ...
- **Cons**: ...
- **Token / Latency Cost**: ...

## 5. Open Questions & Spikes

- [ ] Question 1 (Owner: @user)
- [ ] Question 2 (Owner: @user)

## 6. Discussion Log

*Record chronological summaries of key meetings, comments, or agent debates.*

- **YYYY-MM-DD**: Summary of discussion, disagreements, insights.

## 7. Decision & Rationale

*(Filled when transitioning to `DECIDED`)*

- **Chosen Option**: Option A / Hybrid / etc.
- **Rationale**: Core technical and economic reasons for this decision.
- **Key Trade-offs Accepted**: What concessions were accepted in exchange for this solution.

## 8. Build-Docs Conversion Plan

*(Filled when preparing to graduate into `build-docs/`)*

- [ ] **Target Files**:
    - `build-docs/path/to/target.md` (Update / Create)
- [ ] **Content to Author**:
    - [ ] Canonical specification / current-state design.
    - [ ] Architecture diagrams / schema definitions.
    - [ ] Revision history entry added.
- [ ] **Hygiene Check**:
    - [ ] Verified that past decisions, alternative comparisons, and debate have been excluded from `build-docs`.
    - [ ] Document represents only the pure, prescriptive current state.
- [ ] **Graduation Sign-Off**: Facilitator confirmed and linked to merged build-doc (s).
```

---

## Conversion Pipeline: From Topic to Build-Docs

When a topic gets to agreement (`DECIDED`), use this step-by-step pipeline to convert it and graduate it into [
`build-docs/`](../build-docs/README.md):

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Deliberation in topics/                                  │
│    - Status: IN_DISCUSSION -> DECIDED                       │
│    - Rationale, trade-offs, and decisions finalized         │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Specification Drafting in build-docs/                    │
│    - Translate outcome into prescriptive technical specs    │
│    - Comply with rules: void of past decisions/debate       │
│    - Add revision log entry at bottom of build-doc          │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Graduation & Linkage                                     │
│    - Update topic Status to: GRADUATED                      │
│    - Add pointer in topic to target build-doc(s)            │
│    - Update Topic Registry in topics/topic-registry.md      │
└─────────────────────────────────────────────────────────────┘
```

### Conversion Rules & Quality Standards

1. Remove deliberative language. Do not copy statements like *"We decided on SQLite over Postgres because..."*
   into `build-docs/`. Write this instead: *"The persistence layer uses SQLite with the following schema..."*. The
   historical reasons stay in the topic.
2. Make the specification prescriptive and complete. Make sure that the build-doc contains concrete technical specs
   (interfaces, commands, schemas, metadata tags, directory structures, data flows).
3. Keep traceability. In the `## Revisions` section at the bottom of the build-doc, reference the ID of the source topic:
   ```markdown
   ## Revisions
   | Date         | Version  | Description            | Source Topic      |
   |:-------------|:-------- |:---------------------- |:----------------- |
   |  2026-09-05  | 0.1.0    | Initial specification  | [TOPIC-001](../topics/TOPIC-001_cli-architecture.md)  |
   ```
4. Close the loop. Change the status in the topic file to `GRADUATED`, and link to the generated build-docs.

---

## Build-Docs Domain Mapping Guide

Use this directory map to find where the outcome of a topic belongs in [
`build-docs/`](../build-docs/README.md):

| Domain               | Target Path in `build-docs/`                                             | Scope & Content Type                                                                 |
|:---------------------|:-------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|
| **Overview & Rules** | [`build-docs/README.md`](../build-docs/README.md)                        | Core build-doc rules, high-level project vision.                                     |
| **CLI Binary**       | [`build-docs/cli/`](../build-docs/cli/README.md)                         | CLI command tree, flags, parsing engine, local document querying, checklist manager. |
| **MCP Server**       | [`build-docs/mcp/`](../build-docs/mcp/README.md)                         | Model Context Protocol tools, resources, prompts, agent integration.                 |
| **Static UI**        | [`build-docs/static-ui/`](../build-docs/static-ui/README.md)             | HTML/CSS/JS templates, Material 3 components, single-pane documentation visualizer.  |
| **Work Items**       | [`build-docs/docs/work-items/`](../build-docs/docs/work-items/README.md) | XHTML metadata standards, requirements, designs, phase/task specs, ledger formats.   |
| **Project Wiki**     | [`build-docs/docs/wiki/`](../build-docs/docs/wiki/README.md)             | Coding guidelines, setup guides, architectural standards, repository conventions.    |
| **OpenAPI**          | [`build-docs/docs/open-api/`](../build-docs/docs/open-api/README.md)     | Upstream and downstream service API specs.                                           |

---

## Topic Registry

The [Topic Registry](topic-registry.md) tracks all product ideation topics and architectural decisions in one place.
Read it to see the active, decided, and graduated topics, or to register a new topic.
