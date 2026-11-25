# Intent Decomposition Document Definition

The Intent Decomposition (`02_intent_decomposition.html`) is the implementation plan of a `brevio` work item. It bridges
the accepted Solution Strategy (`01_solution_strategy.html`) and the units that agents converge. It is the reviewed plan
that the build stage checks the diff against.

This document holds the engineering detail that the solution strategy leaves out: component and module changes, data
model and schema changes, and the agent tooling for convergence. It decomposes the intent into a directed acyclic graph
(DAG) of verifiable sub-intents. It enforces dependency ordering and identifies parallel branches. It records the
Protected Resource Registry and catalogs risks and failure modes. A plan view gives the files that change, the order,
the risks, and the proof in one section. A plan interrogation records what the change can break, the riskiest step, and
the options not chosen.

This document serves a dual purpose:

1. **Human Interface:** Renders as a local, fully interactive, static XHTML web page that uses Material Design 3 Web
   Components (`@material/web` / **M3**) and inline DAG diagrams without requiring client-side bundlers or build steps.
2. **Agent / Harness Interface:** Serves as a deterministic, structured XML document queryable via XPath and the `brevio`
   CLI for token-efficient DAG scheduling and sub-intent orchestration.

---

## Core Decomposition Paradigm: Human Engineer vs. AI Coding Tool

In `brevio`, decomposition bridges the gap between human engineering conventions and AI agent requirements across four
dimensions:

| Dimension      | Human Engineer Focus                                                                  | AI Coding Tool Focus                                                                              | `brevio` Decomposition Manifestation                                                                                                     |
|:---------------|:--------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| **Context**    | Relies on tacit codebase conventions and tribal domain knowledge.                     | Needs explicit file paths, referenced patterns, and strict "do not touch" constraints.            | Catalogs target architectural boundaries, links concrete pattern examples, and declares repository-wide protected paths.                |
| **Granularity**| Focuses on high-level patterns and architecture; details left to implementation time. | Requires bounded, single-responsibility goals with deterministic inputs and outputs.              | Partitions the intent into sub-intents with bounded scope, explicit DAG dependencies, and unit count estimates.                          |
| **Verification** | Manual PR review, local exploratory debugging, automated CI.                        | Explicit terminal commands with deterministic output parsing (lint, test, build).                 | Sets a feasibility gate before convergence and a verification command suite for each sub-intent.                                         |
| **Edge Cases** | Usually caught through intuitive testing or code review cycles.                       | Must be itemized upfront to prevent naive happy-path assumptions.                                 | Pre-defines a risk, failure-mode, and rollback matrix that sub-intents and units must satisfy.                                           |

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
3. [Required Sections & M3 Component Structure](#3-required-sections--m3-component-structure)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Plan View](#32-plan-view)
    - 3.3. [Component & Module Changes](#33-component--module-changes)
    - 3.4. [Data Model & Schema Changes](#34-data-model--schema-changes)
    - 3.5. [AI Components & Agent Tooling Specification](#35-ai-components--agent-tooling-specification)
        - 3.5.1. [Assigned & Required Agents](#351-assigned--required-agents)
        - 3.5.2. [Required Skills & Knowledge Packages](#352-required-skills--knowledge-packages)
        - 3.5.3. [AI Coding Tools & MCP Servers](#353-ai-coding-tools--mcp-servers)
        - 3.5.4. [New AI Components Specification](#354-new-ai-components-specification)
    - 3.6. [Decomposition Principles](#36-decomposition-principles)
    - 3.7. [Sub-Intent DAG & Milestone Roadmap](#37-sub-intent-dag--milestone-roadmap)
    - 3.8. [Dependency Graph, Critical Path & Parallel Branches](#38-dependency-graph-critical-path--parallel-branches)
    - 3.9. [Protected Resource Registry](#39-protected-resource-registry)
    - 3.10. [Risk, Failure-Mode & Edge Case Catalog](#310-risk-failure-mode--edge-case-catalog)
    - 3.11. [Plan Interrogation](#311-plan-interrogation)
    - 3.12. [Gating & Orchestration Strategy](#312-gating--orchestration-strategy)
4. [Light Path for a One-Milestone Intent](#4-light-path-for-a-one-milestone-intent)
5. [Authoring & Acceptance](#5-authoring--acceptance)
6. [Token Optimization & XPath Query Patterns](#6-token-optimization--xpath-query-patterns)
7. [Rules & Constraints](#7-rules--constraints)
8. [Revisions](#8-revisions)

---

## 1. File Location & Organization Standard

The intent decomposition document resides at the root of the work item directory, immediately following the solution
strategy:

```text
.brevio/
└── work_items/
    └── {work-item-id}/
        ├── 00_intent_spec.html
        ├── 01_solution_strategy.html
        └── 02_intent_decomposition.html
```

- **File Name:** `02_intent_decomposition.html`
- **Format:** Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- **Encoding:** UTF-8.
- **Runtime:** Fully static, runnable directly in any modern browser via local filesystem (`file://`) or served locally
  via the `brevio` CLI.

---

## 2. XHTML Header & Metadata Standard

Every `02_intent_decomposition.html` begins with universal baseline `<meta>` tags, decomposition tags, sub-intent
metrics, agent tooling tags, and the Material Web ES module importmap loader:

```xhtml

<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>DECOMPOSITION: User Authentication Service</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:02:DECOMP"/>
    <meta name="doc-type" content="intent-decomposition"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="domain" content="work-items"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="status" content="in-progress"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T17:25:00Z"/>
    <meta name="updated-at" content="2026-08-30T17:30:15Z"/>

    <!-- Decomposition Domain Metadata -->
    <meta name="parent-doc-id" content="WI-20260830T170936Z:01:STRATEGY"/>
    <meta name="implements-req" content="EV-001,EV-002,EV-003"/>
    <meta name="risk-class" content="high"/> <!-- routine | high, copied from the strategy -->
    <meta name="layout" content="full"/> <!-- full | light -->
    <meta name="total-sub-intents" content="3"/>
    <meta name="estimated-unit-count" content="9"/>
    <meta name="parallel-branches" content="1"/>
    <meta name="ledger-path" content="03_intent_ledger.json"/>
    <meta name="plan-files" content="db/migrations/**,src/main/java/com/example/auth/**"/>
    <meta name="protected-files" content="src/generated/**,db/migrations/legacy/**"/>
    <meta name="accepted-by" content="human:tech-lead"/> <!-- empty until accepted -->
    <meta name="accepted-at" content="2026-08-30T17:44:00Z"/> <!-- empty until accepted -->

    <!-- AI Components & Tooling Metadata -->
    <meta name="assigned-agents" content="agent:backend-engineer,agent:database-specialist,agent:security-auditor"/>
    <meta name="required-skills" content="skill:spring-boot-jwt-auth,skill:owasp-verification"/>
    <meta name="ai-tools" content="mcp:db-inspector,mcp:ast-grep,cli:brevio"/>
    <meta name="new-ai-components" content="skill:spring-boot-jwt-auth,agent:auth-regression-tester"/>

    <!-- M3 / Material Web Assets -->
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&amp;display=swap" rel="stylesheet"/>
    <link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined" rel="stylesheet"/>
    <script type="importmap">
        {
          "imports": {
            "@material/web/": "https://esm.run/@material/web/"
          }
        }
    </script>
    <script type="module">
        import '@material/web/all.js';
        import {styles as typescaleStyles} from '@material/web/typography/md-typescale-styles.js';

        document.adoptedStyleSheets.push(typescaleStyles.styleSheet);
    </script>
</head>
```

### Intent Decomposition Header Metadata Contracts

| Metadata Tag           | Scope                     | Purpose                                                                       | Example                                            |
|:-----------------------|:--------------------------|:------------------------------------------------------------------------------|:---------------------------------------------------|
| `parent-doc-id`        | Work Item / Decomposition | Points to the parent Solution Strategy `doc-id`                               | `WI-20260830T170936Z:01:STRATEGY`                  |
| `implements-req`       | Work Item / Decomposition | Comma-delimited list of evidence requirement IDs covered                      | `EV-001,EV-002,EV-003`                             |
| `risk-class`           | Work Item / Decomposition | Risk class from the solution strategy. Sets the plan approver role.           | `high`                                             |
| `layout`               | Work Item / Decomposition | `full` for the three-tier layout, `light` for a one-milestone intent          | `full`                                             |
| `total-sub-intents`    | Work Item / Decomposition | Total count of planned sub-intents. The value is `1` for the light path.      | `3`                                                |
| `estimated-unit-count` | Work Item / Decomposition | Total projected intent units across all sub-intents                           | `9`                                                |
| `parallel-branches`    | Work Item / Decomposition | Count of DAG branches that can converge in parallel                           | `1`                                                |
| `ledger-path`          | Work Item / Decomposition | Relative path to the central intent ledger                                    | `03_intent_ledger.json`                            |
| `plan-files`           | Work Item / Decomposition | Comma-delimited union of the Target Scopes of all units                       | `db/migrations/**`                                 |
| `protected-files`      | Work Item / Decomposition | Comma-delimited paths and globs that agents must not change                   | `src/generated/**`                                 |
| `accepted-by`          | Work Item / Decomposition | Identities that accepted the plan                                             | `human:tech-lead`                                  |
| `accepted-at`          | Work Item / Decomposition | Timestamp of the plan acceptance                                              | `2026-08-30T17:44:00Z`                             |
| `assigned-agents`      | Work Item / Decomposition | Agent roles used across convergence                                           | `agent:backend-engineer,agent:database-specialist` |
| `required-skills`      | Work Item / Decomposition | Skill packages required by participating agents                               | `skill:spring-boot-jwt-auth`                       |
| `ai-tools`             | Work Item / Decomposition | MCP servers and CLI tools available to agents during convergence              | `mcp:db-inspector,mcp:ast-grep,cli:brevio`         |
| `new-ai-components`    | Work Item / Decomposition | New AI assets that this plan introduces                                       | `skill:spring-boot-jwt-auth`                       |

---

## 3. Required Sections & M3 Component Structure

The Intent Decomposition document contains 12 standardized sections. Each section combines semantic XHTML elements with
Material Design 3 Web Components (`md-*`), typography scale classes (`md-typescale-*`), and explicit data attributes
(`data-*`) for targeted XPath queries.

### 3.1. Document Header & Metadata Bar

Anchors the document with the work item identifier, decomposition title, lifecycle status chips, and layout. It also
shows the total sub-intents, the estimated unit count, the parent solution strategy link, and the acceptance record.

### 3.2. Plan View

Section id: `plan-view`. The one-read summary of the plan. A reviewer accepts the plan from this section. It has four
parts:

- **Files That Change** (`plan-files`): Each path or glob with its change type (`create`, `modify`, `delete`) and the
  unit that owns it. The list is the union of the Target Scopes of all units, and it matches the `plan-files` tag.
- **Order of Work** (`plan-order`): The sub-intents and units in convergence order, with the parallel waves marked.
- **Risks** (`plan-risks`): The high and critical entries from the Risk Catalog, one line each.
- **Proof** (`plan-proof`): For each evidence requirement (`EV-###`), the command or test that proves it and the unit
  that delivers it.

### 3.3. Component & Module Changes

Section id: `component-changes`. Specifies all source code components, modules, services, repositories, and handlers
that this work item changes or creates. For each module, it defines the file path, responsibility, class and symbol
names, and change type (`create`, `modify`, `deprecate`). Each entry carries a `data-path` attribute.

### 3.4. Data Model & Schema Changes

Section id: `schema-changes`. Captures database migrations, DDL structures, new tables, altered columns, indexing
strategies, and data transfer objects (DTOs). Includes exact schema snippets and entity relationship mappings.

### 3.5. AI Components & Agent Tooling Specification

Section id: `ai-components`. The plan treats the AI agent ecosystem as a first-class architectural layer. This section
specifies the AI agents, skills, and coding tools that take part in the convergence. It also specifies any new AI
components that must be built or configured to deliver the change.

Unit workers query this section via XPath to configure workers, provision skills, and bind MCP servers. The plan
schedules units that create new AI assets before the units that depend on them.

#### 3.5.1. Assigned & Required Agents

Specifies each agent type participating in the convergence, its architectural role, model tier, and operational bounds:

- **Agent Identifier & Role:** Formal agent designation (for example `agent:backend-engineer`,
  `agent:database-specialist`, `agent:security-auditor`, `agent:code-reviewer`).
- **Topology & Delegation:** Hierarchy position (primary worker, verification subagent, or orchestrator) and inter-agent
  communication permissions.
- **Model Tier & Reasoning Profile:** Model tier requirement (`pro` for complex refactoring and ADRs, `flash` for
  targeted file edits, `flash_lite` for quick searches), temperature settings, and reasoning effort.
- **Permissions & Workspace Mode:** Workspace isolation mode (`inherit`, `branch`, `share`), allowed file modification
  paths, and command execution privileges. Each agent holds the least permissions that it needs.
- **Identity Role:** Whether the agent writes units or approves units. The writer identity and the approver identity of a
  unit must differ.

#### 3.5.2. Required Skills & Knowledge Packages

Specifies the existing skill packages and domain instructions that agents must load before converging units:

- **Skill Identifier:** Canonical skill name (for example `skill:spring-boot-jwt-auth`, `skill:owasp-verification`).
- **Source Location:** Path to the skill bundle (for example `.agents/skills/jwt-auth/SKILL.md` or harness builtin).
- **Activation Scope & Triggers:** The sub-intents or unit categories that require this skill.
- **Key Provided Procedures:** Concrete guidelines, code patterns, or validation steps loaded into the agent context.
- **Enforcement Note:** A skill is advisory. A policy that must always hold also needs a `build-gate` or `ci-gate`
  invariant in the solution strategy.

#### 3.5.3. AI Coding Tools & MCP Servers

Details the Model Context Protocol (MCP) servers, CLI tools, AST analyzers, and testing sidecars available to agents:

- **Server / Tool Identifier:** Formal tool provider (for example `mcp:db-inspector`, `mcp:ast-grep`, `cli:brevio`,
  `tool:mvn-test`).
- **Available Operations:** Specific functions exposed to the agent (for example `inspect_schema`, `execute_query`,
  `find_references`).
- **Access Level & Guardrails:** Read-only versus mutating commands, token budget limits, output truncation policies, and
  XPath filter rules.

#### 3.5.4. New AI Components Specification

When the change introduces new AI capabilities, or needs new AI components to converge or maintain the feature, this
subsection specifies them. New AI components must converge in prerequisite units before downstream units rely on them:

1. **New Custom Subagents:**
    - **Name & Purpose:** Unique subagent name (for example `agent:auth-regression-tester`) and target problem domain.
    - **System Prompt Specification:** Core instruction persona, behavioral rules, forbidden actions, and structured
      output contract.
    - **Tool Access Manifest:** Explicit array of permitted tools and MCP capabilities.
    - **Model Tier & Workspace Mode:** Execution model profile and workspace isolation mode (`inherit`, `branch`,
      `share`).

2. **New Custom Skills:**
    - **Skill Identifier & Directory:** Path under `.agents/skills/{skill-slug}/`.
    - **`SKILL.md` Specification:** YAML frontmatter (`name`, `description`), procedural instructions, and execution
      checklists.
    - **Bundled Resources:** Associated helper scripts in `scripts/`, reference templates in `references/`, and asset
      files.
    - **Target Consumer Agents:** Which agents invoke the new skill and under what conditions.

3. **New MCP Tools & Sidecars:**
    - **Tool Name & Namespace:** Unique MCP tool identifier.
    - **JSON Schema Contract:** Exact input argument schemas with property types, descriptions, and required fields.
    - **Output Format:** Return MIME type and structured response schema.
    - **Handler Implementation File:** Source file path implementing the tool logic.

4. **New Hook Templates, Guardrails & Evaluation Rubrics:**
    - Hook templates that enforce `build-gate` invariants, with the hook scope (project settings or managed settings).
    - Automated LLM-as-a-judge evaluation prompts, scoring rubrics, and invariant assertions for unit verification in
      `verification.html`.

### 3.6. Decomposition Principles

Sets the operating rules for converging agents and human reviewers. Articulates how the four dimensions are enforced:

- **Context:** Every unit receives exact file paths and referenced patterns and does not rely on tacit conventions.
- **Granularity:** Every unit is bounded, single-responsibility, and declares a required Target Scope.
- **Verification:** Every stage is verified through explicit terminal commands with deterministic output parsing.
- **Edge Cases:** Known edge cases are addressed in the unit specifications and not discovered by trial and error.

### 3.7. Sub-Intent DAG & Milestone Roadmap

Section id: `sub-intent-breakdown`. Decomposes the intent into sequentially numbered sub-intents. For each sub-intent,
the document records:

- **Sub-Intent Identifier:** For example `sub_intent_01_auth_schema`, `sub_intent_02_jwt_token_service`.
- **Milestone Intent:** A concise statement of the verifiable outcome. The statement declares the outcome and does not
  list implementation steps.
- **Assigned Agent Roles:** Primary converging agent and verification subagents.
- **Estimated Unit Count:** Projected number of intent units.
- **Target Subsystems & Code Boundaries:** Affected modules and source packages.
- **Entrance & Exit Conditions:** Prerequisites to start and verification conditions to finish.

For the light path, this section holds the unit manifest directly. See
[Light Path for a One-Milestone Intent](#4-light-path-for-a-one-milestone-intent).

### 3.8. Dependency Graph, Critical Path & Parallel Branches

Section id: `dependency-graph`. Visualizes and specifies the sub-intent topology:

- **Directed Acyclic Graph (DAG):** Sub-intent dependencies (for example sub-intent 2 depends on sub-intent 1).
- **Parallel Branches:** Branches that can converge concurrently. Two units can run in parallel only if their Target
  Scopes do not overlap and neither unit depends on the other. Units whose Target Scopes overlap run one after another in
  one session. Each parallel branch converges in its own git worktree.
- **Critical Path Nodes:** The sub-intents that set the minimum convergence time.

### 3.9. Protected Resource Registry

Section id: `protected-resources`. Records the repository-wide boundaries that safeguard the codebase against
unchecked drift:

- **Strictly Protected Files:** Paths and packages that agents must not create, modify, or delete (for example core
  infrastructure, generated code, shared schema configuration, and security baseline policies). The list includes every
  `do-not-touch` invariant from the solution strategy, and it also appears in the `protected-files` tag.
- **Frozen Contracts:** Public interfaces, REST API contracts, or DTO structures that must stay backwards-compatible.
- **Resource Limits:** Maximum number of files that one unit can change and token ceilings.
- **Enforcement:** The enforcement level of each protected path. A `build-gate` hook blocks edits to protected paths.

### 3.10. Risk, Failure-Mode & Edge Case Catalog

Section id: `risk-catalog`. Pre-empts naive happy-path convergence by cataloging failure modes:

- **Data Integrity & Migration Risks:** Data loss, migration failures, or schema locking during deployment.
- **System Concurrency & Race Conditions:** Hazards under parallel convergence or high-throughput usage.
- **Agent Drift & Hallucination Vectors:** Code areas prone to subtle syntax or semantic drift, with the invariant that
  guards each area.
- **Rollback & Recovery Procedures:** The rollback command and the conditions that trigger it. The rollback path must be
  rehearsed in a non-production environment before an agent needs it.

### 3.11. Plan Interrogation

Section id: `plan-interrogation`. Records the answers to three questions that the reviewer asks before acceptance:

- **What Can Break** (`data-question="breakage"`): The existing behavior, contracts, and consumers that the change can
  break, with the invariant or test that guards each one.
- **Riskiest Step** (`data-question="riskiest-step"`): The one unit or sub-intent with the highest risk, the reason, and
  the mitigation.
- **Options Not Chosen** (`data-question="alternatives"`): Each implementation option that the plan did not choose, with
  the reason.

### 3.12. Gating & Orchestration Strategy

Section id: `gating-strategy`. Defines the gates that govern transitions:

- **Plan Acceptance Gate:** An adversarial gate first. A reviewing agent with a fresh context reads only this document
  and the solution strategy. It reports whether an engineer who never saw the planning conversation can implement the
  change from the plan alone. Then a human accepts the plan. If `risk-class` is `routine`, an engineer accepts. If
  `risk-class` is `high`, a tech lead or an architect accepts.
- **Feasibility Gate:** `03_intent_verification.html` must record a `pass` verdict for the feasibility check before any
  unit enters `converging`.
- **Sub-Intent Gate:** Sub-intent N+1 unlocks in `03_intent_ledger.json` only when sub-intent N is `verified`, all
  units are `verified`, and `sub_intent_verification.html` records `pass`.
- **Confidence Gates:** The gate type (`deterministic` or `adversarial`) between stages in unattended runs and the named
  human approver for a failed gate.
- **Automated Verification Harness:** The commands that the harness runs between sub-intents.

---

## 4. Light Path for a One-Milestone Intent

An intent that delivers one milestone uses the light path. The `layout` tag is `light`.

- The decomposition declares one implicit sub-intent. `total-sub-intents` is `1`.
- Section `sub-intent-breakdown` holds the unit manifest directly, in the format of the Sequential Atomic Unit Manifest
  in [`sub-intent-specification.md`](sub-intent-specification.md).
- `brevio` writes no `sub_intent_spec.html`, `sub_intent_verification.html`, or `sub_intent_summary.html`.
- Unit directories sit under `units/` at the work item root.
- The ledger keeps one sub-intent entry, so the scheduling rules do not change. The sub-intent gate is the intent
  verification.

```text
.brevio/
└── work_items/
    └── {work-item-id}/
        ├── 00_intent_spec.html
        ├── 01_solution_strategy.html
        ├── 02_intent_decomposition.html
        ├── 03_intent_ledger.json
        ├── ...
        └── units/
            └── unit_001_{slug}/
                ├── intent_unit_spec.html
                └── ...
```

If a second milestone appears during planning, the plan changes to the `full` layout before acceptance.

---

## 5. Authoring & Acceptance

1. The acceptance of `01_solution_strategy.html` starts the decomposition pass. The engineer starts the session in a
   read-only planning mode, gives the agent the intent and the strategy, and asks for the plan.
2. The agent reads the codebase and writes the decomposition, the unit specifications, and the plan view.
3. The engineer interrogates the plan and the agent records the answers in `plan-interrogation`.
4. The engineer iterates until the plan acceptance gate passes.
5. The approver that `risk-class` names accepts the plan. `brevio` records `accepted-by` and `accepted-at`.
6. When the work departs from the plan, the agent updates the plan in the same commit as the change. The plan view and
   the unit Target Scopes stay in step.

---

## 6. Token Optimization & XPath Query Patterns

Orchestrators, subagent workers, and the `brevio` CLI extract targeted execution data using exact XPath expressions
without ingesting the full document.

### Extracting the Plan View

```xpath2
//section[@id='plan-view']
```

### Extracting the Files That Change

```xpath2
/html/head/meta[@name='plan-files']/@content
```

### Extracting the Plan Interrogation

```xpath2
//section[@id='plan-interrogation']//*[@data-question]
```

### Extracting Total Sub-Intent Count & Unit Estimates

```xpath2
/html/head/meta[@name='total-sub-intents']/@content | /html/head/meta[@name='estimated-unit-count']/@content
```

### Querying the Ordered Sub-Intent Breakdown

```xpath2
//section[@id='sub-intent-breakdown']//md-elevated-card[@data-sub-intent-id]
```

### Retrieving the Protected Resource Registry

```xpath2
//section[@id='protected-resources']//md-list-item
```

### Extracting Critical Path Dependencies

```xpath2
//section[@id='dependency-graph']//table[@id='sub-intent-dependency-table']//tr[@data-sub-intent-id]
```

### Querying High-Severity Risks and Mitigations

```xpath2
//section[@id='risk-catalog']//md-outlined-card[@data-risk-severity='high' or @data-risk-severity='critical']
```

### Finding the Units That Touch a Component

```xpath2
//section[@id='component-changes']//*[@data-path]
```

### Extracting the Assigned Agents

```xpath2
/html/head/meta[@name='assigned-agents']/@content
```

---

## 7. Rules & Constraints

1. Every evidence requirement in `01_solution_strategy.html` must map to at least one sub-intent and must appear in
   `plan-proof`.
2. The sub-intent graph must be acyclic.
3. Every sub-intent must declare a milestone outcome and not step-by-step instructions.
4. The `plan-files` tag and the `plan-files` list must equal the union of the Target Scopes of all units.
5. The `protected-files` tag must match the entries in the Protected Resource Registry.
6. Every `do-not-touch` invariant in `01_solution_strategy.html` must appear in the Protected Resource Registry.
7. The `plan-interrogation` section must answer all three questions before the plan is accepted.
8. The plan cannot be accepted until the adversarial plan acceptance gate passes.
9. The acceptance record must name an engineer. If `risk-class` is `high`, it must name a tech lead or an architect.
10. A `light` layout must have exactly one sub-intent.
11. Revision history is tracked exclusively in the `## Revisions` table at the bottom of this document.

---

## 8. Revisions

| Date       | Version | Description                                                                                                                                                                                                  | Source                           |
|:-----------|:--------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|
| 2026-11-25 | 0.3.0   | Made the decomposition the implementation plan. Added the plan view, plan interrogation, plan acceptance gate, `plan-files`, `risk-class`, `layout`, and the acceptance record. Took component, schema, and agent tooling detail from the solution strategy. Added the light path for a one-milestone intent. | TOPIC-007                        |
| 2026-10-31 | 0.2.0   | Replaced the implementation plan with the intent decomposition: sub-intent DAG, parallel branch rules, Protected Resource Registry, confidence gates, and rehearsed rollback.                                  | TOPIC-005                        |
| 2026-09-06 | 0.1.0   | Initial canonical implementation plan specification (`02_implementation_plan.html`) establishing the 4-dimension planning paradigm (Context, Granularity, Validation, Edge Cases), milestone roadmap, global constraints, and XPath extraction patterns. | [Work Item Documents](README.md) |
