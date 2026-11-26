# Sub-Intent Verification Document Definition

The Sub-Intent Verification Document (`sub_intents/{sub_intent_id}/sub_intent_verification.html`) is the authoritative
Tier 2 verification gate in the `brevio` work item lifecycle. It records the automated integration test runs, the rollup
of sub-unit verdicts, and the audits of architecture boundary integrity. It also records the formal gating decisions
(`pass` | `fail` | `warn`). The intent ledger needs these decisions before it unlocks downstream sub-intents.

Sub-intent validation moves verification away from manual human inspection after the fact. The `brevio` harness enforces
deterministic, automated gating in its place.

This document has two uses:

1. Human Interface: The document renders as a local, interactive, static XHTML web page. It uses Material Design 3 Web
   Components (`@material/web`, M3) and needs no client-side bundler or build step. The page shows test pass/fail
   gauges, suite execution logs, invariant checks, and audit badges.
2. Agent / Harness Interface: The document is a deterministic, structured XML document. You can query it with XPath and
   the `brevio` CLI. Tools use it for automated sub-intent transition gating, blocker detection, and remediation
   planning.

---

## Core Paradigm: Applying the Four Dimensions to Sub-Intent Verification

In `brevio`, sub-intent validation enforces strict mechanical checks. It bases verification on the four core dimensions:

| Dimension       | Human Engineer Focus                                                                  | AI Coding Tool Focus                                                                              | `brevio` Sub-Intent Verification Manifestation                                                                                                          |
|:----------------|:--------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------|
| Context         | Uses unwritten codebase conventions and domain knowledge that the team shares.        | Needs explicit file paths, referenced patterns, and strict "do not touch" constraints.            | Runs an automated file boundary audit. Makes sure that the sub-intent changed zero protected or "do not touch" files.                          |
| Granularity     | Focuses on high-level patterns and architecture. Details wait until implementation.   | Needs atomic sub-units with one responsibility each and deterministic inputs and outputs.         | Collects the verdicts of each atomic unit into a full roll-up matrix. Makes sure that 100% of sub-units reached passing status.               |
| Verification    | Manual PR review, local exploratory debugging, automated CI.                          | Explicit terminal commands after each step (lint, test, build), with deterministic parsing of the output. | Runs explicit terminal integration suites, parses process exit codes and assertion counts, and logs raw command outputs deterministically. |
| Edge Cases      | Usually found through intuitive testing or code review cycles.                        | Must be listed in full before the work starts, to prevent naive happy-path assumptions.           | Makes sure that all sub-intent-level boundary edge cases, negative test scenarios, and error-injection suites listed in `sub_intent_spec.html` pass. |

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
3. [Required Sections & M3 Component Structure](#3-required-sections--m3-component-structure)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Sub-Intent Verification Verdict & Gating Decision](#32-sub-intent-verification-verdict--gating-decision)
    - 3.3. [Unit Manifest Execution & Verdict Rollup](#33-unit-manifest-execution--verdict-rollup)
    -
    3.4. [Integration Command Execution & Deterministic Output Parsing](#34-integration-command-execution--deterministic-output-parsing)
    - 3.5. [Boundary Invariant & "Do Not Touch" Compliance Audit](#35-boundary-invariant--do-not-touch-compliance-audit)
    - 3.6. [Sub-Intent Edge Case & Negative Suite Verification](#36-sub-intent-edge-case--negative-suite-verification)
    - 3.7. [Remediation & Defect Resolution Log](#37-remediation--defect-resolution-log)
4. [Token Optimization & XPath Query Patterns](#4-token-optimization--xpath-query-patterns)
5. [Revisions](#5-revisions)

---

## 1. File Location & Organization Standard

Each sub-intent validation document is in the sub-intent directory, next to its parent specification:

```text
.brevio/
└── work_items/
    └── {intent-id}/
        └── sub_intents/
            └── phase_{NN}_{phase_slug}/
                ├── sub_intent_spec.html             # Sub-Intent Scope & Unit Manifest
                ├── sub_intent_verification.html       # Authoritative Sub-Intent Verification Gate
                ├── sub_intent_summary.html          # Sub-Intent Execution Rollup
                └── units/
                    └── ...
```

- File Name: `sub_intent_verification.html`
- Format: Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- Encoding: UTF-8.
- Runtime: Fully static. The file runs directly in any modern browser from the local file system (`file://`), or the
  `brevio` CLI serves it locally.

---

## 2. XHTML Header & Metadata Standard

Every `sub_intent_verification.html` starts with the universal baseline `<meta>` tags, the validation verdict tags, the
test counts, and the Material Web ES module importmap loader:

```xhtml

<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>VALIDATION: Sub-Intent 01 Database Schema &amp; Migration</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:S01:VAL"/>
    <meta name="doc-type" content="sub-intent-verification"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="status" content="completed"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T17:45:00Z"/>
    <meta name="updated-at" content="2026-08-30T17:48:22Z"/>

    <!-- Sub-Intent Verification Domain Metadata -->
    <meta name="sub-intent-id" content="sub_intent_01_database_migration"/>
    <meta name="sub-intent-sequence" content="1"/>
    <meta name="parent-doc-id" content="WI-20260830T170936Z:S01:SPEC"/>
    <meta name="validation-verdict" content="pass"/> <!-- pass | fail | warn -->
    <meta name="tests-passed" content="14"/>
    <meta name="tests-failed" content="0"/>
    <meta name="blocking-issues-count" content="0"/>

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

### Sub-Intent Verification Header Metadata Contracts

| Metadata Tag            | Scope              | Purpose                                               | Example                        |
|:------------------------|:-------------------|:------------------------------------------------------|:-------------------------------|
| `sub-intent-id`              | Sub-Intent / Validation | Unique identifier of the sub-intent directory         | `sub_intent_01_database_migration`  |
| `sub-intent-sequence`        | Sub-Intent / Validation | Number that gives the position of the milestone      | `1`                            |
| `parent-doc-id`         | Sub-Intent / Validation | Points to the `doc-id` of the parent Sub-Intent Specification | `WI-20260830T170936Z:S01:SPEC` |
| `validation-verdict`    | Sub-Intent / Validation | Outcome of the final gating evaluation                | `pass` \| `fail` \| `warn`     |
| `tests-passed`          | Sub-Intent / Validation | Count of passing sub-intent-level integration/system tests | `14`                           |
| `tests-failed`          | Sub-Intent / Validation | Count of failing sub-intent-level integration/system tests | `0`                            |
| `blocking-issues-count` | Sub-Intent / Validation | Count of unresolved defects that block the sub-intent transition | `0`                            |

---

## 3. Required Sections & M3 Component Structure

The Sub-Intent Verification document contains 7 standardized sections. Each section uses semantic XHTML elements and
Material Design 3 Web Components (`md-*`). It also uses typography scale classes (`md-typescale-*`) and explicit data
attributes (`data-*`).

### 3.1. Document Header & Metadata Bar

This section starts the document. It shows the work item identifier, the sub-intent sequence, the sub-intent title, and
the validation verdict badge (`pass`, `fail`, `warn`). It also shows the test counts and a link to the parent sub-intent
specification.

### 3.2. Sub-Intent Verification Verdict & Gating Decision

This section is the executive evaluation that controls the work item lifecycle:

- Formal Verdict: A chip or banner that is easy to see. It shows `PASS` (go to the next sub-intent), `FAIL` (block
  downstream sub-intents and enter remediation), or `WARN` (human review required).
- Gating Decision: A deterministic statement of whether Sub-Intent $N+1$ is authorized to activate in
  `03_intent_ledger.json`.
- Validation Summary: A high-level narrative that summarizes the integration findings, performance changes, and test
  stability.

### 3.3. Unit Manifest Execution & Verdict Rollup

This section collects the status and validation verdicts of all atomic units that `sub_intent_spec.html` defines:

| Unit ID                     | Unit Title                 | Category   | Status      | Validation Verdict | Exit Code | Assertions Passed | Assertions Failed |
|:----------------------------|:---------------------------|:-----------|:------------|:-------------------|:----------|:------------------|:------------------|
| `unit_001_create_tables`    | Create User & Token Tables | `database` | `verified` | `pass`             | `0`       | 8                 | 0                 |
| `unit_002_seed_roles`       | Seed Initial RBAC Roles    | `database` | `verified` | `pass`             | `0`       | 4                 | 0                 |
| `unit_003_verify_migration` | Verify Migration Suite     | `test`     | `verified` | `pass`             | `0`       | 2                 | 0                 |

- The section enforces that zero units stay in `pending`, `in-progress`, or `failed` status.

### 3.4. Integration Command Execution & Deterministic Output Parsing

This section records the end-to-end integration test commands that ran across all sub-intent deliverables:

- Command Line Strings: The exact terminal commands that ran (for example, `mvn verify -Pmigration-tests`).
- Execution Context: The working directory, environment flags, and execution duration.
- Process Exit Code: The exact numeric return code (`0` required for pass).
- Parsed Output Metrics: Test counts, execution times, memory overhead, and status lines that a regex matched.
- Console Snippets: Raw stdout and stderr logs that the reader can expand, for diagnostic inspection.

### 3.5. Boundary Invariant & "Do Not Touch" Compliance Audit

This section is an automated compliance check of repository integrity:

- Protected File Audit: Asserts that zero files in the sub-intent or global "do not touch" manifests changed.
- Schema & Interface Invariant Assertions: Makes sure that backward compatibility and schema integrity hold.
- Git Tree Audit: Makes sure that git status is clean and that all changed files are in authorized directories.

### 3.6. Sub-Intent Edge Case & Negative Suite Verification

This section records the tests of all edge cases and boundary hazards that `sub_intent_spec.html` listed:

- Negative Input Injection: Makes sure that the code correctly rejects malformed data or unauthorized requests and
  handles the errors.
- Concurrency & Race Condition Checks: Make sure that multi-threaded or parallel execution keeps its integrity.
- Rollback Drills: Assert that migration down-scripts or rollback procedures are idempotent.

### 3.7. Remediation & Defect Resolution Log

If a sub-intent verification verdict is `fail` or `warn`, this section records the specific remediation approach:

- Identified Defect: The exact failure description and the unit or file that caused it.
- Root Cause Analysis: A technical explanation of why the failure occurred.
- Required Remediation Steps: The atomic unit assignments that are sent out to correct the fault before sub-intent
  validation runs again.

---

## 4. Token Optimization & XPath Query Patterns

Orchestrators, harness lifecycle runners, and reporting agents use targeted XPath to query specific validation nodes:

### Querying Sub-Intent Verification Verdict & Blocking Issues

```xpath2
/html/head/meta[@name='validation-verdict']/@content | /html/head/meta[@name='blocking-issues-count']/@content
```

### Retrieving Failing Sub-Units across the Sub-Intent

```xpath2
//table[@id='unit-rollup-table']//tr[@data-validation-verdict='fail']
```

### Extracting Integration Test Output and Exit Codes

```xpath2
//section[@id='integration-commands']//div[@data-command-exit-code!='0']
```

### Checking "Do Not Touch" Boundary Violations

```xpath2
//section[@id='boundary-audit']//md-list-item[@data-audit-status='violation']
```

### Querying Active Remediation Units

```xpath2
//section[@id='remediation-log']//md-elevated-card[@data-remediation-status='open']
```

---

## 5. Revisions

| Date       | Version | Description                                                                                                                                                                                              | Source                           |
|:-----------|:--------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|
| 2026-11-25 | 0.2.1   | Rewrote the prose in plain English. No change to the design. | User request |
| 2026-10-31 | 0.2.0 | Moved to the intent-based document set: intent, sub-intent, and unit terminology, convergence states, required target scope, and evidence requirement IDs. | TOPIC-005 |
| 2026-09-06 | 0.1.0   | Initial canonical sub-intent validation specification (`sub_intent_verification.html`) defining automated gating verdicts, unit rollups, integration command parsing, boundary compliance audits, and XPath queries. | [Work Item Documents](README.md) |
