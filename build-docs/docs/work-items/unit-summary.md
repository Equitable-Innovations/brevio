# Unit Summary Document Definition

The Unit Summary Document (`units/{unit_id}/summary.html`) is the authoritative Tier 3 post-execution record in the
`brevio` work item lifecycle. It records the real outcomes of an atomic unit: the exact source code files created and
modified, the unified git diff metrics, and the commit references. It also records the verification assertions from
`verification.html` and the token use telemetry.

The unit summary closes the loop on atomic unit execution. It gives an immutable audit trail from the declared goal to
the code commits.

This document has two uses:

1. Human Interface: The document renders as a local, interactive, static XHTML web page. It uses Material Design 3 Web
   Components (`@material/web`, M3) and needs no client-side bundler or build step. The page shows diff statistics,
   file touchpoint cards, commit metadata, and telemetry badges.
2. Agent / Harness Interface: The document is a deterministic, structured XML document. You can query it with XPath and
   the `brevio` CLI. Tools use it for automated ledger updates, sub-intent rollups, and PR preparation.

---

## Core Paradigm: Applying the Four Dimensions to Unit Summaries

In `brevio`, unit summaries record execution compliance against the four basic dimensions:

| Dimension       | Human Engineer Focus                                                                  | AI Coding Tool Focus                                                                              | `brevio` Unit Summary Manifestation                                                                                                           |
|:----------------|:--------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|
| Context         | Uses unwritten codebase conventions and domain knowledge that the team shares.        | Needs explicit file paths, referenced patterns, and strict "do not touch" constraints.            | Formally lists the exact files created, modified, and deleted. Makes sure that no protected or "do not touch" file has an unauthorized change. |
| Granularity     | Focuses on high-level patterns and architecture. Details wait until implementation.   | Needs atomic sub-units with one responsibility each and deterministic inputs and outputs.         | Keeps code changes and git diff metrics strictly inside the bounded responsibility of the atomic unit. No scope pollution crosses units.    |
| Verification    | Manual PR review, local exploratory debugging, automated CI.                          | Explicit terminal commands after each step (lint, test, build), with deterministic parsing of the output. | Shows deterministic verification evidence: process exit codes, test pass rates, lint compliance, and changes in automated test coverage. |
| Edge Cases      | Usually found through intuitive testing or code review cycles.                        | Must be listed in full before the work starts, to prevent naive happy-path assumptions.           | Audits execution against the edge case catalog that the plan listed (`EC-###`). Makes sure that the work satisfied all negative scenarios and boundary tests. |

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
3. [Required Sections & M3 Component Structure](#3-required-sections--m3-component-structure)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Unit Execution Outcome & Verdict Summary](#32-unit-execution-outcome--verdict-summary)
    - 3.3. [File Modification Manifest & Diff Metrics](#33-file-modification-manifest--diff-metrics)
    - 3.4. [Git Commit & Repository State](#34-git-commit--repository-state)
    - 3.5. [Verification & Assertion Rollup](#35-verification--assertion-rollup)
    - 3.6. [Context & "Do Not Touch" Compliance Sign-off](#36-context--do-not-touch-compliance-sign-off)
    - 3.7. [Edge Case Satisfaction Audit](#37-edge-case-satisfaction-audit)
    - 3.8. [Token & Execution Telemetry Summary](#38-token--execution-telemetry-summary)
4. [Token Optimization & XPath Query Patterns](#4-token-optimization--xpath-query-patterns)
5. [Revisions](#5-revisions)

---

## 1. File Location & Organization Standard

Each unit summary document is directly inside its isolated unit directory, next to its specification and validation
documents:

```text
.brevio/
└── work_items/
    └── {intent-id}/
        └── sub_intents/
            └── phase_{NN}_{phase_slug}/
                └── units/
                    └── task_{NNN}_{task_slug}/
                        ├── intent_unit_spec.html          # Unit Goal, Scope & Verification Contract
                        ├── verification.html         # Test Execution & Verification Record
                        ├── intent.diff               # Authoritative Intent Diff
                        ├── summary.html            # Authoritative Unit Output & Diffs
                        ├── conversation.jsonl      # Agent Interaction Log
                        └── metrics.json            # Token & Latency Telemetry
```

- File Name: `summary.html`
- Format: Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- Encoding: UTF-8.
- Runtime: Fully static. The file runs directly in any modern browser from the local file system (`file://`), or the
  `brevio` CLI serves it locally.

---

## 2. XHTML Header & Metadata Standard

Every `summary.html` starts with the universal baseline `<meta>` tags, the unit outcome tags, the file paths, and the
Material Web ES module importmap loader:

```xhtml

<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>SUMMARY: Unit 001 Create User &amp; Token Tables</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:S01:U001:SUM"/>
    <meta name="doc-type" content="unit-summary"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="sub-intent-id" content="sub_intent_01_database_migration"/>
    <meta name="unit-id" content="unit_001_create_tables"/>
    <meta name="parent-doc-id" content="WI-20260830T170936Z:S01:U001:SPEC"/>
    <meta name="status" content="completed"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T18:00:00Z"/>
    <meta name="updated-at" content="2026-08-30T18:02:15Z"/>

    <!-- Unit Summary Domain Metadata -->
    <meta name="unit-verdict" content="success"/> <!-- success | failed -->
    <meta name="execution-duration-ms" content="26200"/>
    <meta name="files-created" content="src/main/resources/db/migration/V1__init_auth.sql"/>
    <meta name="files-modified" content=""/>
    <meta name="commit-hash" content="a86d7c2ca972b77f0e540ed6a84d395c6647dd6d"/>
    <meta name="diff-ref" content="intent.diff"/>
    <meta name="metrics-ref" content="metrics.json"/>
    <meta name="conversation-ref" content="conversation.jsonl"/>

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

### Unit Summary Header Metadata Contracts

| Metadata Tag            | Scope          | Purpose                                                | Example                             |
|:------------------------|:---------------|:-------------------------------------------------------|:------------------------------------|
| `unit-id`               | Unit / Summary | Unique identifier of the unit directory                | `unit_001_create_tables`            |
| `sub-intent-id`              | Unit / Summary | Identifier of the parent sub-intent directory     | `sub_intent_01_database_migration`       |
| `parent-doc-id`         | Unit / Summary | Points to the `doc-id` of the parent Intent Unit Specification | `WI-20260830T170936Z:S01:U001:SPEC` |
| `unit-verdict`          | Unit / Summary | Class of the final outcome of unit execution           | `success` \| `failed`               |
| `execution-duration-ms` | Unit / Summary | Total elapsed time for unit execution, in milliseconds | `26200`                             |
| `files-created`         | Unit / Summary | Comma-delimited relative paths of new source files     | `src/db/V1__init.sql`               |
| `files-modified`        | Unit / Summary | Comma-delimited relative paths of edited source files  | `src/app/User.java`                 |
| `commit-hash`           | Unit / Summary | SHA of the git commit that contains the code changes of the atomic unit | `a86d7c2ca972b77f...`               |
| `diff-ref`              | Unit / Summary | Relative path to the unified diff patch of the unit    | `intent.diff`                         |
| `metrics-ref`           | Unit / Summary | Relative path to the token metrics JSON of the unit    | `metrics.json`                      |
| `conversation-ref`      | Unit / Summary | Relative path to the conversation JSONL of the unit    | `conversation.jsonl`                |

---

## 3. Required Sections & M3 Component Structure

The Unit Summary document contains 8 standardized sections. Each section uses semantic XHTML elements and Material
Design 3 Web Components (`md-*`). It also uses typography scale classes (`md-typescale-*`) and explicit data attributes
(`data-*`).

### 3.1. Document Header & Metadata Bar

This section starts the document. It shows the work item ID, the sub-intent ID, the unit ID, and the execution outcome
chip (`success`, `failed`). It also shows the commit hash chip, the duration, and a navigation link back to
`intent_unit_spec.html`.

### 3.2. Unit Execution Outcome & Verdict Summary

This section is the executive summary of the completed atomic work:

- Verdict Banner: A high-contrast indicator that shows `SUCCESS` or `FAILED`.
- Solution Narrative: A summary of the specific technical solution that the unit delivered, the symbols it created, and
  the schemas it applied.
- Precondition vs. Postcondition State: Proof that the file system reached the expected postconditions.

### 3.3. File Modification Manifest & Diff Metrics

This section gives a detailed breakdown of the source code changes:

| File Path                                           | Change Type | Lines Added | Lines Deleted | Net Delta | Language |
|:----------------------------------------------------|:------------|:------------|:--------------|:----------|:---------|
| `src/main/resources/db/migration/V1__init_auth.sql` | `CREATED`   | +48         | -0            | +48       | SQL      |
| `src/test/resources/fixtures/auth_seed.sql`         | `MODIFIED`  | +12         | -2            | +10       | SQL      |

- Diff View: Syntax-highlighted unified diff snippets that show the main structural changes.

### 3.4. Git Commit & Repository State

This section records the version control metadata for the change:

- Git Commit Hash: The full 40-character SHA and the short hash.
- Commit Message: A formatted commit message that follows the project commit standards (for example,
  `feat(auth): add users and refresh_tokens migration`).
- Working Tree Cleanliness: An assertion that the git working tree was clean after the commit.

### 3.5. Verification & Assertion Rollup

This section summarizes the validation results that `verification.html` records:

- Process Exit Code: Makes sure that all terminal commands returned exit code `0`.
- Assertion Scorecard: The total assertions evaluated, passed, and failed (`8 passed, 0 failed`).
- Code Coverage Delta: The net change in test coverage that the unit caused (`+2.4%`).

### 3.6. Context & "Do Not Touch" Compliance Sign-off

This section audits and proves that the work stayed inside its context boundaries:

- Boundary Compliance: A statement that zero files outside `target-files` were touched.
- "Do Not Touch" Sign-off: An assertion that all protected files and frozen API signatures stayed untouched.

### 3.7. Edge Case Satisfaction Audit

This section compares each edge case that `intent_unit_spec.html` listed (`EC-001` through `EC-004`) with the verified
outcomes:

- A statement that duplicate keys, null tokens, cascade deletes, and timezone offsets behave as the specification
  states.

### 3.8. Token & Execution Telemetry Summary

This section links the execution cost and latency telemetry:

- Duration: Wall-clock execution time, in milliseconds.
- Tokens Used: The prompt, completion, and cache tokens used.
- Estimated Cost: The API cost, in USD.
- Telemetry Links: Links to [`intent.diff`](intent-diff.md), [`metrics.json`](unit-conversation-metrics.md), and
  [`conversation.jsonl`](unit-conversation.md).

---

## 4. Token Optimization & XPath Query Patterns

Downstream orchestrators, ledger synchronizers, and PR authoring agents use targeted XPath to query specific summary
fragments:

### Querying Unit Verdict & Commit Hash

```xpath2
/html/head/meta[@name='unit-verdict']/@content | /html/head/meta[@name='commit-hash']/@content
```

### Retrieving Created and Modified File Manifests

```xpath2
/html/head/meta[@name='files-created']/@content | /html/head/meta[@name='files-modified']/@content
```

### Extracting Diff Line Statistics

```xpath2
//table[@id='file-diff-table']//tr[@data-file-path]
```

### Checking "Do Not Touch" Compliance Sign-off

```xpath2
//section[@id='boundary-compliance']//md-list-item[@data-compliance-status='verified']
```

### Extracting Edge Case Verification Status

```xpath2
//section[@id='edge-case-audit']//md-list-item[@data-edge-case-status='satisfied']
```

---

## 5. Revisions

| Date       | Version | Description                                                                                                                                                                                                              | Source                           |
|:-----------|:--------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|
| 2026-11-25 | 0.3.1   | Rewrote the prose in plain English. No change to the design. | User request |
| 2026-10-31 | 0.3.0 | Moved to the intent-based document set: intent, sub-intent, and unit terminology, convergence states, required target scope, and evidence requirement IDs. | TOPIC-005 |
| 2026-09-06 | 0.2.0   | Integrated `intent.diff` reference in file structure, metadata header (`diff-ref`), and artifact links.                                                                                                                    | [Work Item Documents](README.md) |
| 2026-09-06 | 0.1.0   | Initial canonical unit summary specification (`summary.html`) detailing the 4-dimension paradigm (context diffs, atomic boundaries, validation rollups, edge case satisfaction), git commit tracking, and XPath queries. | [Work Item Documents](README.md) |
