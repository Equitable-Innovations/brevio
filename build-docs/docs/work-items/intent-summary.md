# Intent Summary Document Definition

The Intent Summary Document (`04_intent_summary.html`) is the authoritative Tier 1 summary that `brevio` writes after
execution. It covers the full lifecycle of the work item. It collects all completed sub-intents, atomic units, validation
verdicts, source code diffs, and the total token telemetry into one completion report for executives. The report
certifies that the feature is delivered and authorizes the pull request.

The intent summary is the high-level audit trail. It proves that the implementation follows the evidence requirements
and the design in `01_solution_strategy.html` exactly and safely.

This document has two uses:

1. Human Interface: The document renders as a local, interactive, static XHTML web page. It uses Material Design 3 Web
   Components (`@material/web`, M3) and needs no client-side bundler or build step. The page shows a delivery
   dashboard for executives, evidence traceability matrices, sub-intent milestone rollups, total diff statistics, and a
   PR description generator.
2. Agent / Harness Interface: The document is a deterministic, structured XML document. You can query it with XPath and
   the `brevio` CLI. Tools use it to create PRs automatically, compile release notes, and index the work item archive.

---

## Core Paradigm: Applying the Four Dimensions to Intent Summary

In `brevio`, the intent summary makes sure that the full implementation meets the four basic dimensions:

| Dimension       | Human Engineer Focus                                                                  | AI Coding Tool Focus                                                                              | `brevio` Intent Summary Manifestation                                                                                                                |
|:----------------|:--------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
| Context         | Uses unwritten codebase conventions and domain knowledge that the team shares.        | Needs explicit file paths, referenced patterns, and strict "do not touch" constraints.            | Lists all file changes in the repository, audits compliance with the repo-wide "do not touch" boundaries, and records the branch and PR metadata. |
| Granularity     | Focuses on high-level patterns and architecture. Details wait until implementation.   | Needs atomic sub-units with one responsibility each and deterministic inputs and outputs.         | Summarizes the milestones and keeps a link down to every atomic unit, commit, and conversation transcript.                                       |
| Verification    | Manual PR review, local exploratory debugging, automated CI.                          | Explicit terminal commands after each step (lint, test, build), with deterministic parsing of the output. | Collects the verification evidence for the full lifecycle: pre-execution intent verification, all sub-intent gates that passed, regression suites, and 0 failing tests. |
| Edge Cases      | Usually found through intuitive testing or code review cycles.                        | Must be listed in full before the work starts, to prevent naive happy-path assumptions.           | Gives a post-mortem audit that makes sure that the work satisfied all risks, boundary edge cases, and failure modes that the plan listed.        |

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
3. [Required Sections & M3 Component Structure](#3-required-sections--m3-component-structure)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Work Item Completion Verdict & Executive Synthesis](#32-work-item-completion-verdict--executive-synthesis)
    - 3.3. [Evidence Fulfillment & Traceability Verification](#33-evidence-fulfillment--traceability-verification)
    - 3.4. [Sub-Intent & Milestone Execution Rollup](#34-sub-intent--milestone-execution-rollup)
    -
    3.5. [Cumulative Code Deliverables & File Modification Registry](#35-cumulative-code-deliverables--file-modification-registry)
    - 3.6. [Full-Lifecycle Verification & Quality Gate Audit](#36-full-lifecycle-verification--quality-gate-audit)
    - 3.7. [Edge Case, Risk & Failure-Mode Post-Mortem](#37-edge-case-risk--failure-mode-post-mortem)
    - 3.8. [Comprehensive Telemetry & Cost Accounting](#38-comprehensive-telemetry--cost-accounting)
    - 3.9. [Pull Request & Deployment Handoff Guide](#39-pull-request--deployment-handoff-guide)
4. [Token Optimization & XPath Query Patterns](#4-token-optimization--xpath-query-patterns)
5. [Revisions](#5-revisions)

---

## 1. File Location & Organization Standard

The intent summary document is at the root of the work item directory:

```text
.brevio/
└── work_items/
    └── {intent-id}/
        ├── 00_intent_spec.html
        ├── 01_solution_strategy.html
        ├── 02_intent_decomposition.html
        ├── 03_intent_ledger.json
        ├── 03_intent_verification.html
        ├── 04_intent_summary.html                # Authoritative Work Item Summary
        ├── 05_intent_conversation_summary.html
        ├── 06_intent_conversation_metrics.json
        └── sub_intents/
            └── ...
```

- File Name: `04_intent_summary.html`
- Format: Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- Encoding: UTF-8.
- Runtime: Fully static. The file runs directly in any modern browser from the local file system (`file://`), or the
  `brevio` CLI serves it locally.

---

## 2. XHTML Header & Metadata Standard

Every `04_intent_summary.html` starts with the universal baseline `<meta>` tags, the intent outcome tags, the rollup
metrics, and the Material Web ES module importmap loader:

```xhtml

<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>INTENT SUMMARY: User Authentication Service</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:04:SUM"/>
    <meta name="doc-type" content="intent-summary"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="status" content="completed"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T18:15:00Z"/>
    <meta name="updated-at" content="2026-08-30T18:18:40Z"/>

    <!-- Intent Summary Domain Metadata -->
    <meta name="parent-doc-id" content="WI-20260830T170936Z:02:DECOMP"/>
    <meta name="execution-duration-ms" content="312000"/>
    <meta name="completed-sub-intents" content="3"/>
    <meta name="completed-units" content="9"/>
    <meta name="final-verdict" content="success"/> <!-- success | partial | failed -->
    <meta name="total-tokens" content="98400"/>
    <meta name="total-cost-usd" content="0.2952"/>
    <meta name="pr-branch" content="feature/user-auth-service"/>
    <meta name="ledger-path" content="03_intent_ledger.json"/>

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

### Intent Summary Header Metadata Contracts

| Metadata Tag            | Scope               | Purpose                                                   | Example                            |
|:------------------------|:--------------------|:----------------------------------------------------------|:-----------------------------------|
| `parent-doc-id`         | Work Item / Summary | Points to the `doc-id` of the parent Intent Decomposition | `WI-20260830T170936Z:02:DECOMP`      |
| `execution-duration-ms` | Work Item / Summary | Total elapsed time from the start of the first unit to release sign-off | `312000`                           |
| `completed-sub-intents`      | Work Item / Summary | Total count of sub-intents that completed successfully  | `3`                                |
| `completed-units`       | Work Item / Summary | Total count of atomic units that ran, across all sub-intents | `9`                                |
| `final-verdict`         | Work Item / Summary | Class of the overall convergence outcome                  | `success` \| `partial` \| `failed` |
| `total-tokens`          | Work Item / Summary | Total token use across all sub-intents and units          | `98400`                            |
| `total-cost-usd`        | Work Item / Summary | Total money spent on LLM API use, in USD                  | `0.2952`                           |
| `pr-branch`             | Work Item / Summary | Git branch that contains all committed deliverables       | `feature/user-auth-service`        |
| `ledger-path`           | Work Item / Summary | Relative path to the intent ledger                        | `03_intent_ledger.json`    |

---

## 3. Required Sections & M3 Component Structure

The Intent Summary document contains 9 standardized sections. Each section uses semantic XHTML elements and Material
Design 3 Web Components (`md-*`). It also uses typography scale classes (`md-typescale-*`) and explicit data attributes
(`data-*`).

### 3.1. Document Header & Metadata Bar

This section starts the document. It shows the work item ID, the feature title, and the final outcome chip (`success`,
`partial`, `failed`). It also shows the counts of completed sub-intents and units, the total wall-clock duration, and a
link to the parent intent decomposition.

### 3.2. Work Item Completion Verdict & Executive Synthesis

This section is the executive summary. It certifies that the work item is complete:

- Final Verdict Banner: A banner that is easy to see. It shows `SUCCESSFUL DELIVERY`.
- Executive Narrative: A full, short summary of the business value and the architecture components that the work
  delivered.
- Merge Readiness Certification: A clear statement that all quality gates and tests passed.

### 3.3. Evidence Fulfillment & Traceability Verification

This section audits the delivery of each evidence requirement from `01_solution_strategy.html`:

| Evidence ID | Evidence Description                      | Delivering Sub-Intent       | Delivering Units       | Verification Verdict | RFC 2119 Level |
|:----------|:-------------------------------------------|:-----------------------|:-----------------------|:---------------------|:---------------|
| `EV-001` | User password authentication with Argon2id | `sub_intent_01`, `sub_intent_02` | `unit_001`, `unit_004` | `PASS`               | `MUST`         |
| `EV-002` | JWT issue and validation                   | `sub_intent_02`             | `unit_005`, `unit_006` | `PASS`               | `MUST`         |
| `EV-003` | Refresh token rotation                     | `sub_intent_01`, `sub_intent_02` | `unit_002`, `unit_007` | `PASS`               | `SHOULD`       |

- The section enforces 100% evidence traceability. No evidence requirement is orphaned.

### 3.4. Sub-Intent & Milestone Execution Rollup

This section is a scorecard for each completed sub-intent:

| Sub-Intent ID                      | Sub-Intent Title                    | Sequence | Status      | Units | Duration | Tokens | Verdict |
|:------------------------------|:-------------------------------|:---------|:------------|:------|:---------|:-------|:--------|
| `sub_intent_01_database_migration` | Database Schema & Migration    | 1        | `verified` | 3/3   | 94.5s    | 24,500 | `pass`  |
| `sub_intent_02_service_layer`      | Authentication Service & JWT   | 2        | `verified` | 4/4   | 142.1s   | 48,200 | `pass`  |
| `sub_intent_03_rest_endpoints`     | REST Auth Endpoints & Security | 3        | `verified` | 2/2   | 75.4s    | 25,700 | `pass`  |

### 3.5. Cumulative Code Deliverables & File Modification Registry

This section lists all code changes at the repository level:

- Total Files Created / Modified: The total count (for example, 6 created, 4 modified, 0 deleted).
- Net Lines of Code: The total lines added (+540) and lines removed (-32).
- Subsystem Breakdown: The changes, grouped by layer (database, service, API, tests).

### 3.6. Full-Lifecycle Verification & Quality Gate Audit

This section collects the evidence from all verification gates:

- Feasibility Check: Makes sure that `03_intent_verification.html` recorded a feasibility `pass` before convergence
  started.
- Sub-Intent-Level Integration Gates: Makes sure that all sub-intent validation documents recorded `pass`.
- End-to-End Regression Suite: Makes sure that the full test suite has zero test failures.
- Security & Vulnerability Audit: Makes sure that the change introduced no security alerts or OWASP violations.

### 3.7. Edge Case, Risk & Failure-Mode Post-Mortem

This section reviews how the work handled the system risks and edge cases from `02_intent_decomposition.html`:

- A statement that automated tests proved the edge cases for token expiration, concurrent refresh requests, and DB
  failover.

### 3.8. Comprehensive Telemetry & Cost Accounting

This section shows in detail the resources that the agents used:

- Total Execution Duration: 312,000ms (~5.2 minutes).
- Aggregated Token Usage: 98,400 tokens across all agents that took part.
- Total Expenditure: $0.2952 USD.
- Efficiency Metric: The ratio of tokens to lines, and the cache hit rates.

### 3.9. Pull Request & Deployment Handoff Guide

This section helps the review and the merge:

- Git Branch: `feature/user-auth-service`.
- Generated PR Description: A Markdown template that summarizes the intent, the delivered components, and the
  verification evidence.
- Deployment & Migration Order: The order of the migrations and of the service rollout.

---

## 4. Token Optimization & XPath Query Patterns

Downstream CI/CD pipelines, release note generators, and CLI commands use these queries to extract the high-level
summary metrics:

### Querying Final Verdict & Execution Duration

```xpath2
/html/head/meta[@name='final-verdict']/@content | /html/head/meta[@name='execution-duration-ms']/@content
```

### Retrieving Evidence Completion Status

```xpath2
//table[@id='evidence-traceability-table']//tr[@data-evidence-id]
```

### Extracting Sub-Intent Milestone Rollup

```xpath2
//table[@id='sub-intent-rollup-table']//tr[@data-sub-intent-id]
```

### Querying Total Token Consumption and Cost

```xpath2
/html/head/meta[@name='total-tokens']/@content | /html/head/meta[@name='total-cost-usd']/@content
```

### Retrieving Generated PR Description Template

```xpath2
//section[@id='pr-handoff-guide']//code[@id='pr-description-markdown']
```

---

## 5. Revisions

| Date       | Version | Description                                                                                                                                                                                            | Source                           |
|:-----------|:--------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|
| 2026-11-25 | 0.3.1 | Rewrote the prose in plain English. No change to the design. | User request |
| 2026-11-25 | 0.3.0 | Evidence requirements now come from the solution strategy. | TOPIC-007 |
| 2026-10-31 | 0.2.0 | Moved to the intent-based document set: intent, sub-intent, and unit terminology, convergence states, required target scope, and evidence requirement IDs. | TOPIC-005 |
| 2026-09-06 | 0.1.0   | Initial canonical intent summary specification (`04_intent_summary.html`) establishing work item executive synthesis, requirement traceability audits, sub-intent rollups, cumulative diffs, and XPath queries. | [Work Item Documents](README.md) |
