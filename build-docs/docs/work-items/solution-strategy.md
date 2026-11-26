# Solution Strategy Document Definition

The Solution Strategy Document (`01_solution_strategy.html`) is the requirements and design specification in the `brevio`
work item lifecycle. An agent writes it from the accepted `00_intent_spec.html`, under the organization's skills for
brand, security, compliance, and UX. It declares what the solution must do: the requirements, the system invariants, the
boundary envelope, the evidence requirements, and the autonomy tier. It also declares the design: the solution
hypothesis, interface and contract changes, flagged concerns, and micro-architectural decision records (ADRs). An
evidence traceability matrix links the two parts.

The product owner reviews and accepts the strategy, but does not write it. The strategy names no source files, symbols,
or schema statements. That detail belongs to the [Intent Decomposition](intent-decomposition.md)
(`02_intent_decomposition.html`), which engineers plan against the accepted strategy.

This document has two purposes:

1. Human Interface: The document renders as a local, fully interactive, static XHTML web page. The page uses Material
   Design 3 Web Components (`@material/web` / M3) and inline diagrams, and needs no client-side bundler and no build step.
2. Agent / Harness Interface: The document is a deterministic, structured XML document. Decomposition and convergence
   agents query it through XPath and the `brevio` CLI, and they read it with few tokens.

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
3. [Required Sections & M3 (Material Web) Component Structure](#3-required-sections--m3-material-web-component-structure)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Solution Hypothesis & Architecture Diagram](#32-solution-hypothesis--architecture-diagram)
    - 3.3. [Requirements](#33-requirements)
    - 3.4. [System Invariants & Preservation](#34-system-invariants--preservation)
    - 3.5. [Boundary Envelope](#35-boundary-envelope)
    - 3.6. [Evidence Requirements](#36-evidence-requirements)
    - 3.7. [Open Question Resolution](#37-open-question-resolution)
    - 3.8. [Interface & Contract Changes](#38-interface--contract-changes)
    - 3.9. [Flagged Concerns](#39-flagged-concerns)
    - 3.10. [Technical Decisions & Trade-offs (Micro-ADRs)](#310-technical-decisions--trade-offs-micro-adrs)
    - 3.11. [Evidence Traceability Matrix](#311-evidence-traceability-matrix)
4. [Authoring & Acceptance](#4-authoring--acceptance)
5. [Token Optimization & XPath Query Patterns](#5-token-optimization--xpath-query-patterns)
6. [Rules & Constraints](#6-rules--constraints)
7. [Revisions](#7-revisions)

---

## 1. File Location & Organization Standard

The solution strategy document is at the root of the work item directory, directly after the intent specification:

```text
.brevio/
└── work_items/
    └── {work-item-id}/
        ├── 00_intent_spec.html
        └── 01_solution_strategy.html
```

- File Name: `01_solution_strategy.html`
- Format: Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- Encoding: UTF-8.
- Runtime: Fully static. The file runs directly in any modern browser from the local filesystem (`file://`), or the
  `brevio` CLI serves it locally.

---

## 2. XHTML Header & Metadata Standard

Every `01_solution_strategy.html` begins with universal baseline `<meta>` tags, strategy-specific tags, provenance tags,
and the Material Web ES module importmap loader:

```xhtml

<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>STRATEGY: User Authentication Service</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:01:STRATEGY"/>
    <meta name="doc-type" content="solution-strategy"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="domain" content="work-items"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="status" content="completed"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T17:15:00Z"/>
    <meta name="updated-at" content="2026-08-30T17:22:15Z"/>

    <!-- Strategy Domain Metadata -->
    <meta name="parent-doc-id" content="WI-20260830T170936Z:00:SPEC"/>
    <meta name="autonomy-tier" content="propose"/> <!-- observe | propose | change-with-review | scoped-automation -->
    <meta name="risk-class" content="high"/> <!-- routine | high -->
    <meta name="invariants-hash" content="sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"/>
    <meta name="requirement-count" content="4"/>
    <meta name="invariant-count" content="3"/>
    <meta name="evidence-count" content="3"/>
    <meta name="open-concerns" content="0"/>
    <meta name="architectural-domain"
          content="backend-service"/> <!-- backend-service | database | full-stack | cli | core-lib -->
    <meta name="target-frameworks" content="spring-boot,postgresql,jwt,argon2"/>
    <meta name="accepted-by" content="human:product-owner,human:tech-lead"/> <!-- empty until accepted -->
    <meta name="accepted-at" content="2026-08-30T17:43:00Z"/> <!-- empty until accepted -->

    <!-- Provenance Metadata -->
    <meta name="generated-by" content="agent:strategy-author@v1.0"/>
    <meta name="prompt-ref" content="sha256:2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"/>
    <meta name="policy-versions" content="secure-api-review@1.2.0,brand-guidelines@3.0.1"/>

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

### Provenance Metadata Contracts

| Metadata Tag      | Purpose                                                                                     | Example                                    |
|:------------------|:--------------------------------------------------------------------------------------------|:-------------------------------------------|
| `generated-by`    | Identity of the agent that wrote the strategy                                               | `agent:strategy-author@v1.0`               |
| `prompt-ref`      | `sha256:` hash of the prompt that produced the strategy                                     | `sha256:2c26...e7ae`                       |
| `policy-versions` | Comma-delimited policy skills and the versions that were in force for the strategy pass     | `secure-api-review@1.2.0`                  |

---

## 3. Required Sections & M3 (Material Web) Component Structure

The Solution Strategy document contains 11 standardized sections. Each section combines semantic XHTML elements with
Material Design 3 Web Components (`md-*`) and typography scale classes (`md-typescale-*`). Each section also has
explicit data attributes (`data-*`) for targeted XPath queries.

### 3.1. Document Header & Metadata Bar

Anchors the document with the work item identifier, strategy title, status, and parent intent reference. It also shows
the autonomy tier, the risk class, the open concern count, the evidence count, and the acceptance record.

### 3.2. Solution Hypothesis & Architecture Diagram

Section id: `solution-hypothesis`. Outlines the high-level approach and the module interaction flow in an inline SVG or
Mermaid diagram. Shows the boundary between new components, changed services, external integrations, and AI agent
touchpoints. It names components and services, not files or classes.

### 3.3. Requirements

Section id: `requirements`. States the behavior and the user-facing outcome that the solution delivers. Each requirement
is an `md-list-item` with a `data-requirement-id` attribute (for example `REQ-001`) and a `data-source` attribute. The
source names the section of `00_intent_spec.html` that the requirement comes from (for example `proposed-outcome`). A
requirement that applies a brand, security, compliance, or UX policy names the policy skill and its version.

### 3.4. System Invariants & Preservation

Section id: `invariants`. Each invariant is an `md-list-item` with a `data-invariant-id` attribute (for example
`INV-001`), a `data-class` attribute, and a `data-level` attribute. Each entry records:

- A description of the condition that must hold.
- A shell command. If the invariant holds, the command exits with code `0`.
- The enforcement level: `advisory`, `build-gate`, `ci-gate`, or `human-gate`.
- The class: `must-preserve`, `do-not-touch`, `performance-budget`, or `security-boundary`.
- The components at risk and the protective mechanism that keeps the invariant true.

A `do-not-touch` invariant feeds the Protected Resource Registry in `02_intent_decomposition.html`.

### 3.5. Boundary Envelope

Section id: `boundary-envelope`. Lists the capabilities that are in scope and the capabilities that are strictly out of
scope (`data-scope="in"` or `data-scope="out"`). Lists the constraints that bind the solution, for example the
authentication method that the intent must reuse. Every constraint in `00_intent_spec.html` appears here with its
`data-constraint-id`.

### 3.6. Evidence Requirements

Section id: `evidence-requirements`. Each evidence requirement has these parts:

- A `data-evidence-id` attribute (for example `EV-001`).
- The requirement IDs that it proves.
- A description of the observable outcome.
- A verification command with a pass or fail result.

Downstream documents bind to these identifiers through the `implements-req` tag. If any evidence requirement has no
`pass` verdict, the intent cannot move to `settled`.

### 3.7. Open Question Resolution

Section id: `open-question-resolution`. Lists every `data-question-id` from `00_intent_spec.html`. Each entry has a
`data-status` attribute:

- `resolved`: The entry records the answer.
- `carried-forward`: The entry names the owner and the document or stage that answers the question.

### 3.8. Interface & Contract Changes

Section id: `interface-changes`. Specifies public APIs, REST endpoints, CLI commands, or cross-module RPC interfaces.
Combines HTTP method chips (`GET`, `POST`, `PUT`, `DELETE`), URL path definitions, request parameters, JSON schema
contracts, and response status examples.

### 3.9. Flagged Concerns

Section id: `flagged-concerns`. Lists each area of concern that the strategy pass found, for example two policies that
contradict each other. Each concern is an `md-list-item` with a `data-concern-id` attribute and a description. It also
names the policy skills involved, the policy owner, and a status (`open`, `resolved`). Before the strategy is accepted,
the product owner resolves each concern with its policy owner.

### 3.10. Technical Decisions & Trade-offs (Micro-ADRs)

Section id: `technical-decisions`. Documents the isolated architectural and technical decisions for this work item only. Each micro-ADR is an `md-outlined-card` with a `data-adr-id` attribute (for example `ADR-001`) and records:

- Context and problem statement.
- Decision taken.
- Options not chosen, with the reason for each.
- Trade-offs (pros and cons).
- Impact on downstream convergence and agent workflows.

### 3.11. Evidence Traceability Matrix

Section id: `evidence-traceability`. Cross-references each evidence requirement ID (`EV-###`) with the requirements,
invariants, interfaces, and micro-ADRs in this document:

| Evidence ID | Requirements | Invariants | Interface / Contract      | Micro-ADR |
|:------------|:-------------|:-----------|:--------------------------|:----------|
| `EV-001`    | `REQ-001`    | `INV-001`  | `POST /api/v1/auth/login` | `ADR-001` |
| `EV-002`    | `REQ-002`    | `INV-002`  | `Bearer <token>` Header   | `ADR-002` |

`02_intent_decomposition.html` extends this traceability down to components, sub-intents, and units.

---

## 4. Authoring & Acceptance

1. The acceptance of `00_intent_spec.html` starts the strategy pass. A non-interactive job can run it.
2. An agent reads `00_intent_spec.html` with the organization's policy skills loaded and writes the strategy. The agent
   flags every concern, especially where two policies contradict each other.
3. The product owner reviews the strategy against the intent. Does it solve the stated problem? Does it answer or carry
   forward every open question?
4. The product owner resolves each flagged concern with its policy owner.
5. The product owner accepts the strategy. If `risk-class` is `high`, a tech lead accepts it too. `brevio` records
   `accepted-by` and `accepted-at`.
6. The acceptance starts the decomposition pass.

---

## 5. Token Optimization & XPath Query Patterns

Decomposition agents (`02_intent_decomposition.html`) and unit workers query specific sections with exact XPath 1.0/2.0
expressions. Thus they do not read the full document. This keeps the token overhead low and reduces model
hallucinations.

| Extraction Objective            | XPath Query Pattern                                                                 |
|:--------------------------------|:------------------------------------------------------------------------------------|
| Requirements                    | `//section[@id='requirements']//md-list-item`                                       |
| All invariants                  | `//section[@id='invariants']//md-list-item`                                         |
| Invariants that block edits     | `//section[@id='invariants']//md-list-item[@data-level='build-gate']`               |
| Evidence requirements           | `//section[@id='evidence-requirements']//md-list-item`                              |
| Out-of-scope capabilities       | `//section[@id='boundary-envelope']//*[@data-scope='out']`                          |
| Carried-forward questions       | `//section[@id='open-question-resolution']//*[@data-status='carried-forward']`      |
| Open concerns                   | `//section[@id='flagged-concerns']//md-list-item[@data-status='open']`              |
| Micro-ADRs                      | `//section[@id='technical-decisions']//md-outlined-card`                            |
| Autonomy tier                   | `/html/head/meta[@name='autonomy-tier']/@content`                                   |

---

## 6. Rules & Constraints

1. Every requirement must name its source section in `00_intent_spec.html`.
2. Every constraint and every open question in `00_intent_spec.html` must appear in this document.
3. Every invariant must have a shell command and an enforcement level.
4. Every evidence requirement must be a command with a pass or fail result and must appear in the Evidence Traceability
   Matrix.
5. The `invariants-hash` tag must match a hash of the `invariants` section. A change to an invariant updates the hash.
6. The strategy cannot be accepted while a flagged concern has the status `open`.
7. The strategy names components, services, and contracts. It must not name source files, classes, symbols, or schema
   statements, and it must not contain step-by-step edit instructions.
8. The acceptance record must name the product owner. If `risk-class` is `high`, it must also name a tech lead.
9. This document tracks its revision history only in the `## Revisions` table at the bottom of this document.

---

## 7. Revisions

| Date       | Version | Description                                                                                                                                                                                                                                                          | Source                           |
|:-----------|:--------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|
| 2026-11-25 | 0.5.1   | Rewrote the prose in plain English. No change to the design. | User request |
| 2026-11-25 | 0.5.0   | Made the strategy the requirements and design specification. Added requirements, open question resolution, `prompt-ref`, `risk-class`, and the acceptance record. Took the invariants, boundary envelope, evidence requirements, and autonomy tier from the intent specification. Moved component, schema, and agent tooling detail to the intent decomposition. Concerns now block acceptance. Micro-ADRs record options not chosen. | TOPIC-007                        |
| 2026-10-31 | 0.4.0   | Replaced the technical design with the solution strategy: solution hypothesis, invariant preservation strategy, flagged concerns, hook templates, writer and approver identity roles, and evidence traceability matrix.                                              | TOPIC-005                        |
| 2026-09-06 | 0.3.0   | Added AI Components & Agent Tooling specification (assigned agents, required skills, AI coding tools/MCP servers, and new AI components architecture), updated header metadata contract, traceability matrix, XPath patterns, and complete XHTML reference template. | Work Item Architecture Alignment |
| 2026-09-06 | 0.2.0   | Removed the XHTML snippets as the UI defined ones generated in the initial draft were junk.                                                                                                                                                                          | [Work Item Documents](README.md) |
