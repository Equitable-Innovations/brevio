# Intent Verification Document Definition

The Intent Verification Document (`03_intent_verification.html`) belongs to the work item lifecycle of the Token
Optimized Software Harness (`brevio`). It is the authoritative Tier 1 master verification contract and gating authority
of that lifecycle. It sets the full verification criteria, test suites, deterministic command assertions, and quality
standards for the work item. The work item must prove all of them before it is certified as `settled` and ready for the
pull request merge. The document also makes sure that every declared system invariant holds.

[`01_solution_strategy.html`](solution-strategy.md) defines *what* requirements, invariants, and evidence the work item
must achieve. [`02_intent_decomposition.html`](intent-decomposition.md) defines *how* the execution is partitioned into
milestones. `03_intent_verification.html` defines *how the implemented code is proven correct*. Code delivery is thus not
subjective. Deterministic commands, test pyramids, non-functional quality gates, edge-case validations, and explicit
manual sign-offs verify it.

`03_intent_verification.html` has a lifecycle with two stages:

1. Feasibility Stage (Master Verification Plan & Feasibility Gate): This stage is written upfront during
   `formulating`, before any unit converges. It sets the complete verification strategy. It maps 100% of evidence
   requirements, invariants, and acceptance criteria to deterministic verification commands. It also defines the
   multi-level test pyramid and catalogs negative/edge-case tests. Before the harness moves the intent to `converging`,
   this stage makes sure that the intent decomposition itself is complete, sound, and fully verifiable.
2. Completion Stage (Final Completion Gate & Roll-up Audit): This stage is updated as each sub-intent completes. When all
   sub-intents finish, the stage is finalized. It executes the work-item-wide end-to-end suites. It audits the rollup of
   all Tier 2 Sub-Intent Validations ([`sub_intent_verification.html`](sub-intent-verification.md)) and Tier 3 Unit
   Verifications ([`verification.html`](unit-verification.md)). It makes sure that the evidence and acceptance criteria
   are fulfilled, runs every invariant command, and records manual sign-offs. Then it gives the authoritative completion
   verdict (`pass` | `fail` | `warn`). This verdict authorizes [`04_intent_summary.html`](intent-summary.md) generation
   and pull request submission.

This document has two purposes:

1. Human Interface (Static UI): The document renders as a local, fully interactive, static XHTML web page with
   Material Design 3 Web Components (`@material/web` / M3). Engineering leads, QA specialists, and reviewers inspect
   test pass gauges, evidence verification matrices, command execution logs, and sign-off checklists directly in any
   browser. The page needs no client-side bundler and no runtime server.
2. Agent / Harness Interface (Machine Parseable): The document is a deterministic, structured XML document. Agents
   query it through XPath and the `brevio` CLI. The harness mechanically enforces this document as a hard gate. If the
   feasibility check fails, worker agents cannot start a unit. The intent cannot reach `settled` in
   [`03_intent_ledger.json`](intent-ledger.md) until the final completion verdict is `pass` and a named human
   authorizes the release.

---

## Core Paradigm: Applying the Four Dimensions to Intent Verification

In `brevio`, intent verification replaces retrospective, informal human spot-checks with deterministic gating that is
defined upfront. It applies the four core dimensions:

| Dimension       | Human Engineer Focus                                                                  | AI Coding Tool Focus                                                                              | `brevio` Intent Verification Manifestation                                                                                                                                                        |
|:----------------|:--------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Context**     | Relies on tacit codebase conventions and tribal domain knowledge.                     | Needs explicit file paths, referenced patterns, and strict "do not touch" constraints.            | Formally catalogs test environments, databases, mock services, test fixtures, environment variables, and protected repository file boundaries ("do not touch") required for test execution. |
| **Granularity** | Focuses on high-level patterns and architecture; details left to implementation time. | Requires atomic, single-responsibility sub-units with deterministic inputs/outputs.               | Structures validation into a multi-level testing pyramid: Tier 3 atomic unit/unit assertions, Tier 2 sub-intent integration suites, and Tier 1 work-item-wide end-to-end user journeys.          |
| **Verification** | Manual PR review, local exploratory debugging, automated CI.                          | Explicit terminal commands with deterministic output parsing (lint, test, build) after each step. | Mandates verbatim command lines, expected numeric exit codes (`0`), stdout/stderr regex patterns, assertion count thresholds, and explicit manual sign-off protocols.                       |
| **Edge Cases**  | Usually caught through intuitive testing or code review cycles.                       | Must be exhaustively itemized upfront to prevent naive happy-path assumptions.                    | Pre-defines exhaustive negative test cases, adversarial input injection, security boundary scans, rate-limit thresholds, concurrency race checks, and fault injection tests.                |

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
    - 2.1. [XHTML Header Example](#21-xhtml-header-example)
    - 2.2. [Metadata Attribute Contracts](#22-metadata-attribute-contracts)
3. [Required Sections & M3 Component Structure](#3-required-sections--m3-component-structure)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Executive Verification Strategy & Completion Definition](#32-executive-verification-strategy--completion-definition)
    - 3.3. [Evidence & Acceptance Criteria Verification Matrix](#33-evidence--acceptance-criteria-verification-matrix)
    - 3.4. [Multi-Level Testing Pyramid & Verification Command Suites](#34-multi-level-testing-pyramid--verification-command-suites)
        - 3.4.1. [Tier 3 Unit & Component Verification Suites](#341-tier-3-unit--component-verification-suites)
        - 3.4.2. [Tier 2 Integration & API Contract Suites](#342-tier-2-integration--api-contract-suites)
        - 3.4.3. [Tier 1 Work-Item-Wide End-to-End (E2E) & System Suites](#343-tier-1-work-item-wide-end-to-end-e2e--system-suites)
    - 3.5. [Non-Functional, Security, Invariant & Code Quality Gates](#35-non-functional-security-invariant--code-quality-gates)
    - 3.6. [Edge Case, Negative & Boundary Verification](#36-edge-case-negative--boundary-verification)
    - 3.7. [Sub-Intent & Unit Verification Roll-up Audit](#37-sub-intent--unit-verification-roll-up-audit)
    - 3.8. [Manual Verification & Human Sign-Off Protocol](#38-manual-verification--human-sign-off-protocol)
    - 3.9. [Defect Remediation Log & Gating Verdict](#39-defect-remediation-log--gating-verdict)
4. [Token Optimization & XPath Query Patterns](#4-token-optimization--xpath-query-patterns)
5. [Lineage, Rollup & Ledger Integration](#5-lineage-rollup--ledger-integration)
6. [Rules & Constraints](#6-rules--constraints)
7. [Revisions](#7-revisions)

---

## 1. File Location & Organization Standard

The intent verification document is at the root of the work item directory, next to the core Tier 1 artifacts:

```text
.brevio/
└── work_items/
    └── {YYYYMMDDTHHMMSSZ}_{work_item_slug}/
        ├── 00_intent_spec.html                # Originator's Intent (Frozen at Acceptance)
        ├── 01_solution_strategy.html                 # Requirements, Invariants, Evidence & Design
        ├── 02_intent_decomposition.html         # Implementation Plan & Sub-Intent DAG
        ├── 03_intent_ledger.json       # Convergence State Machine & Unit DAG
        ├── 03_intent_verification.html             # Authoritative Intent Verification Gate
        ├── 04_intent_summary.html                # Final Work Item Execution Summary
        ├── 05_intent_conversation_summary.html   # Distilled LLM Formulation Dialogue
        ├── 06_intent_conversation_metrics.json   # Formulation Telemetry & Token Metrics
        ├── 07_review_findings.html            # Review Findings by Pass and Severity
        └── sub_intents/
            └── ...
```

- File Name: `03_intent_verification.html`
- Format: Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- Encoding: UTF-8.
- Runtime: Fully static. The file runs directly in modern browsers from the local filesystem (`file://`), or the
  `brevio` CLI serves it locally.
- Paired Artifacts:
    - Intent Specification: [
      `00_intent_spec.html`](intent-specification.md)
    - Solution Strategy: [
      `01_solution_strategy.html`](solution-strategy.md)
    - Intent Decomposition: [
      `02_intent_decomposition.html`](intent-decomposition.md)
    - Intent Ledger: [
      `03_intent_ledger.json`](intent-ledger.md)
    - Sub-Intent Verifications: `sub_intents/{sub_intent_id}/sub_intent_verification.html`
    - Unit Verifications: `units/{unit_id}/verification.html`
    - Execution Rollup: [
      `04_intent_summary.html`](intent-summary.md)

---

## 2. XHTML Header & Metadata Standard

### 2.1. XHTML Header Example

Every `03_intent_verification.html` begins with universal baseline `<meta>` tags, validation lifecycle state, test
statistics, and the Material Web ES module importmap loader:

```xhtml
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>INTENT VERIFICATION: User Authentication Service</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:03:VAL"/>
    <meta name="doc-type" content="intent-verification"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="domain" content="work-items"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="status" content="completed"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T17:40:00Z"/>
    <meta name="updated-at" content="2026-08-30T18:14:20Z"/>

    <!-- Intent Verification Domain Metadata -->
    <meta name="parent-doc-id" content="WI-20260830T170936Z:02:DECOMP"/>
    <meta name="validation-stage"
          content="final-completion-verdict"/> <!-- feasibility | final-completion-verdict -->
    <meta name="validation-verdict" content="pass"/> <!-- pass | fail | warn | pending -->
    <meta name="validation-scope" content="completeness,feasibility,security,acceptance,e2e"/>
    <meta name="tests-planned" content="42"/>
    <meta name="tests-executed" content="42"/>
    <meta name="tests-passed" content="42"/>
    <meta name="tests-failed" content="0"/>
    <meta name="tests-skipped" content="0"/>
    <meta name="blocking-issues-count" content="0"/>
    <meta name="fpsr" content="1.0"/> <!-- 1.0 (pass on first attempt) | 0.0 (required rework) -->
    <meta name="manual-verification-required" content="true"/>
    <meta name="manual-sign-off-status" content="approved"/> <!-- pending | approved | rejected | n/a -->

    <!-- Material Design 3 Web Component Loader -->
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
<body>
<main class="brevio-doc-container">
    <!-- Structured Document Body -->
</main>
</body>
</html>
```

### 2.2. Metadata Attribute Contracts

| Metadata Tag                   | Scope                  | Purpose                                                              | Example                                            |
|:-------------------------------|:-----------------------|:---------------------------------------------------------------------|:---------------------------------------------------|
| `parent-doc-id`                | Work Item / Verification | Points to parent Intent Decomposition `doc-id`                        | `WI-20260830T170936Z:02:DECOMP`                      |
| `validation-stage`             | Work Item / Verification | Lifecycle stage of the verification document                           | `feasibility` \| `final-completion-verdict` |
| `validation-verdict`           | Work Item / Verification | Formal validation gating verdict                                     | `pass` \| `fail` \| `warn` \| `pending`            |
| `validation-scope`             | Work Item / Verification | Comma-delimited list of evaluated verification domains               | `completeness,feasibility,security,evidence,e2e,invariants` |
| `tests-planned`                | Work Item / Verification | Total number of test assertions and cases defined in the verification plan            | `42`                                               |
| `tests-executed`               | Work Item / Verification | Number of test assertions/cases executed to date                     | `42`                                               |
| `tests-passed`                 | Work Item / Verification | Total number of passing test assertions                              | `42`                                               |
| `tests-failed`                 | Work Item / Verification | Total number of failing test assertions                              | `0`                                                |
| `tests-skipped`                | Work Item / Verification | Number of intentionally skipped test cases with justification        | `0`                                                |
| `blocking-issues-count`        | Work Item / Verification | Tally of open defects preventing the feasibility pass or completion         | `0`                                                |
| `fpsr`                         | Work Item / Verification | First-Pass Success Rate (`1.0` if passed without remediation rework) | `1.0` \| `0.0`                                     |
| `manual-verification-required` | Work Item / Verification | Indicates whether human sign-off items exist in the checklist        | `true` \| `false`                                  |
| `manual-sign-off-status`       | Work Item / Verification | Status of required manual verification items                         | `pending` \| `approved` \| `rejected` \| `n/a`     |

---

## 3. Required Sections & M3 Component Structure

The Intent Verification document contains 9 standardized sections. The sections use semantic XHTML elements and
Material Design 3 Web Components (`md-*`). They also use typography scale classes (`md-typescale-*`) and explicit data
attributes (`data-*`).

### 3.1. Document Header & Metadata Bar

Anchors the document with the work item identification and the verification stage badge (`feasibility` or
`final-completion-verdict`). It shows the validation verdict chip (`pass`, `fail`, `warn`) and the test pass gauge. It
also
shows the blocking issue counter and navigation links back to [`01_solution_strategy.html`](solution-strategy.md) and
[`02_intent_decomposition.html`](intent-decomposition.md).


### 3.2. Executive Verification Strategy & Completion Definition

States the overall verification philosophy. It also sets the clear, objective definition of "100% complete" for the
work item:

- Definition of Done (DoD): Explicit checklist of mandatory conditions. All conditions must be met before coding is
  certified complete:
    1. 100% of evidence requirements in [
       `01_solution_strategy.html`](solution-strategy.md) and all unit acceptance criteria pass
       automated verification.
    2. All Tier 2 Sub-Intent Verifications ([
       `sub_intent_verification.html`](sub-intent-verification.md))
       report `validation-verdict="pass"` with zero blocking issues.
    3. All Tier 3 Unit Verifications ([
       `verification.html`](unit-verification.md))
       report `validation-verdict="pass"` and clean exit code `0`.
    4. Work-item-wide end-to-end integration and system test suite passes with zero regressions.
    5. Every invariant command exits with code `0`, and zero protected "do not touch" file boundaries are violated.
    6. Code quality, linting, formatting, and security vulnerability scans pass with zero errors.
    7. All mandatory manual verification checklist items are formally signed off.
- Verification Governance: The `brevio` harness executes deterministic gating rules. A single failing test or
  unhandled evidence requirement halts completion and triggers the automated remediation loop.

### 3.3. Evidence & Acceptance Criteria Verification Matrix

The authoritative 1:1 traceability registry. It maps every evidence requirement ID from
[`01_solution_strategy.html`](solution-strategy.md) and every unit acceptance criterion to concrete verification
commands, automated test assertions, and pass/fail evidence:

- Completeness Invariant: If any evidence requirement ID from `01_solution_strategy.html` is absent from this matrix,
  `03_intent_verification.html` fails the feasibility check with `blocking-issues-count >= 1`.

### 3.4. Multi-Level Testing Pyramid & Verification Command Suites

Structures verification across the testing pyramid. It details verbatim terminal commands, execution contexts, and
deterministic output parsing rules:

#### 3.4.1. Tier 3 Unit & Component Verification Suites

- Purpose: Makes sure that atomic functions, domain models, utility classes, and validation logic are correct in total
  isolation.
- Mocking Policy: External dependencies (database, network, message queues) must be mocked or stubbed.
- Verification Command: Verbatim terminal invocation (for example `mvn test -Dtest=*UnitTest`, `pytest tests/unit/`,
  `npm run test:unit`).
- Assertion Criteria: 100% assertions pass, line coverage meets project threshold (for example $\ge 80\%$), zero
  uncaught exceptions.

#### 3.4.2. Tier 2 Integration & API Contract Suites

- Purpose: Makes sure that cross-component interaction, database migrations, persistence repositories, and HTTP/REST
  contract boundaries are correct.
- Environment: Embedded test database (Testcontainers PostgreSQL, H2) or local mock server.
- Verification Command: Verbatim terminal invocation (for example `mvn verify -Pintegration-tests`,
  `pytest tests/integration/`).
- Assertion Criteria: HTTP status codes match OpenAPI contracts, database transactions commit/rollback accurately,
  schema constraints enforced.

#### 3.4.3. Tier 1 Work-Item-Wide End-to-End (E2E) & System Suites

- Purpose: Makes sure that the entire user journey across multiple sub-intents is correct. It exercises complete
  end-to-end workflows in an integrated environment.
- Verification Command: Verbatim terminal invocation (for example `npm run test:e2e`, `mvn verify -Pe2e-tests`).
- Execution Evidence: Detailed terminal output, assertions passed, and latency metrics:

### 3.5. Non-Functional, Security, Invariant & Code Quality Gates

Makes sure that the code meets architectural, security, and maintainability baselines, and that every declared invariant
holds:

- Invariant Monitors: One row per invariant (`INV-###`) from `01_solution_strategy.html`. Each row lists the shell command,
  the enforcement level (`advisory`, `build-gate`, `ci-gate`, `human-gate`), the exit code, and the literal output.
  A non-zero exit code on a `ci-gate` invariant blocks a `pass` verdict. A `human-gate` invariant holds the ledger state
  until the named approver signs.
- Evidence from the Toolchain: The literal output of each command is the evidence. The agent summary is not evidence.
- Gate Type: The gate is `deterministic`, `adversarial`, or `human`. An adversarial gate runs a reviewing agent in a
  fresh context.

### 3.6. Edge Case, Negative & Boundary Verification

Records verification of the negative paths, error handling, adversarial inputs, and edge cases cataloged in [
`02_intent_decomposition.html`](intent-decomposition.md):

| Edge Case ID | Category       | Scenario / Adversarial Input                   | Expected System Behavior                                         | Verifying Test Case                         | Verdict |
|:-------------|:---------------|:-----------------------------------------------|:-----------------------------------------------------------------|:--------------------------------------------|:--------|
| `EDGE-001`   | Authentication | Replay attack with expired refresh token       | Entire refresh token family revoked. HTTP 401 emitted.           | `TokenRotationE2ETest#testReplayAttack`     | `PASS`  |
| `EDGE-002`   | Input Boundary | Password length = 0 or > 128 characters        | HTTP 400 Bad Request with field-specific validation error        | `UserValidationTest#testPasswordBoundaries` | `PASS`  |
| `EDGE-003`   | Concurrency    | Simultaneous registration with identical email | Exactly one user record created. Second request returns HTTP 409. | `ConcurrencyRegistrationTest#testRace`      | `PASS`  |
| `EDGE-004`   | Infrastructure | Database connection timeout during auth        | HTTP 503 Service Unavailable. No unhandled stack traces leaked.  | `FaultInjectionTest#testDbTimeout`          | `PASS`  |

### 3.7. Sub-Intent & Unit Verification Roll-up Audit

Evaluates the roll-up integrity across the execution hierarchy of the work item. For `03_intent_verification.html` to
reach a `pass` verdict, all underlying sub-intent and unit verifications must pass.

- Roll-up Rule: If any sub-intent verification reports `fail` or `warn`, or any unit verification has exit code
  $\ne 0$, the intent verification verdict cannot be `pass`.

### 3.8. Manual Verification & Human Sign-Off Protocol

Covers the verification steps that require human sensory inspection, exploratory testing, UX review, or compliance
sign-off.

- If `manual-verification-required="true"`, any item in `pending` or `rejected` status blocks final intent verification
  approval.

### 3.9. Defect Remediation Log & Gating Verdict

The formal gating verdict that authorizes work item transitions, together with the defect remediation history:

- Formal Verdict: High-visibility chip or banner that shows one of these values:
    - `PASS`: All automated suites, quality gates, invariants, rollups, and manual items passed. The intent can move to
      `verified`, and a named human can authorize [`04_intent_summary.html`](intent-summary.md) generation and the
      release.
    - `FAIL`: One or more blocking issues detected. The harness places a hold with a named human approver and
      dispatches targeted remediation back to worker agents.
    - `WARN`: Non-blocking advisories detected (for example a minor performance deviation or an optional warning).
      Progress is conditional and allowed upon human acknowledgement.
- Defect Remediation History: Itemizes any defects found during the feasibility or completion stage, with the root
  cause, remediation commits, and re-test verification evidence.

---

## 4. Token Optimization & XPath Query Patterns

Downstream CI/CD runners, orchestrator agents, and release automation extract verification data from
`03_intent_verification.html` with targeted XPath queries. They do not load the entire document into context:

### Querying Verification Verdict & Stage

```xpath2
/html/head/meta[@name='validation-verdict']/@content | /html/head/meta[@name='validation-stage']/@content
```

### Checking Blocking Issue Count & Test Counts

```xpath2
/html/head/meta[@name='blocking-issues-count']/@content | /html/head/meta[@name='tests-passed']/@content | /html/head/meta[@name='tests-failed']/@content
```

### Retrieving Failing Evidence Requirements in the Traceability Matrix

```xpath2
//table[@id='evidence-matrix-table']//tr[@data-status='fail']
```

### Extracting E2E Verification Command & Output

```xpath2
//section[@id='e2e-command-suite']//code | //section[@id='e2e-command-suite']//pre
```

### Verifying Quality Gate and Invariant Compliance

```xpath2
//table[@id='quality-gates-table']//tr[@data-status!='pass'] | //table[@id='invariant-monitors-table']//tr[@data-status!='pass']
```

### Auditing Manual Sign-Off Status

```xpath2
/html/head/meta[@name='manual-sign-off-status']/@content | //table[@id='manual-checks-table']//tr[@data-status='pending']
```

---

## 5. Lineage, Rollup & Ledger Integration

The intent verification document anchors the entire verification lifecycle from formulation to pull request authorization:

```mermaid
graph TD
    REQ["01_solution_strategy.html<br/>(Requirements, Invariants & Evidence Requirements)"] -->|" 1:1 Traceability "| VAL["03_intent_verification.html<br/>(Master Verification Spec & Completion Gate)"]
    PLAN["02_intent_decomposition.html<br/>(Sub-Intent DAG, Protected Resources & Edge Cases)"] -->|" Test Pyramid & Gates "| VAL
    VAL -->|" Feasibility Gate: PASS "| LEDGER["03_intent_ledger.json<br/>(Unlocks Sub-Intent 1 Execution)"]

    subgraph Execution Hierarchy
        P1["Sub-Intent 1 Execution"] --> P1V["sub_intents/sub_intent_01/sub_intent_verification.html"]
        P2["Sub-Intent 2 Execution"] --> P2V["sub_intents/sub_intent_02/sub_intent_verification.html"]
        P3["Sub-Intent 3 Execution"] --> P3V["sub_intents/sub_intent_03/sub_intent_verification.html"]
    end

    P1V -->|" Roll-up Tier 2 "| VAL
    P2V -->|" Roll-up Tier 2 "| VAL
    P3V -->|" Roll-up Tier 2 "| VAL
    VAL -->|" Final Completion Gate: PASS "| SUM["04_intent_summary.html<br/>(Authorizes PR Submission)"]
```

- Feasibility Gating: The harness reads `/html/head/meta[@name='validation-verdict']/@content` in
  `03_intent_verification.html`. If the value is not `pass` (or a conditional `warn`), sub-intent convergence stays
  mechanically locked in `03_intent_ledger.json`.
- Hierarchical Roll-up: All Tier 3 unit verifications roll up into their parent Tier 2 sub-intent verifications. All
  sub-intent verifications roll up into `03_intent_verification.html`.
- Final Completion Gate: When all sub-intents complete and the work-item-wide E2E suites pass, `validation-verdict` is
  finalized as `pass`. This verdict authorizes [`04_intent_summary.html`](intent-summary.md) generation.

---

## 6. Rules & Constraints

1. 100% Evidence Coverage Invariant: Every evidence requirement defined in
   [`01_solution_strategy.html`](solution-strategy.md) and every unit acceptance criterion must have a corresponding
   entry in the Evidence & Acceptance Criteria Verification Matrix. Zero unverified evidence requirements are permitted.
2. Deterministic Command Execution: All automated test entries must specify exact, runnable terminal commands with
   deterministic expected exit codes (`0`) and stdout/stderr assertions. Generic phrases like "run test suite" are
   strictly forbidden.
3. Dual-Stage Lifecycle Preservation: The document must record both its feasibility baseline and its final
   completion verdict. The transition from `feasibility` to `final-completion-verdict` must reflect actual executed test
   runs.
   3a. Invariant Coverage: Every invariant in `01_solution_strategy.html` must have a row in the Invariant Monitors
   table.
   3b. Evidence Integrity: Each verdict must cite the literal command output.
4. Zero-Tolerance Defect Gating: Any failing test, unresolved blocker, or unapproved mandatory manual check strictly
   prevents `validation-verdict` from reaching `pass`. A failed gate must name a human approver.
5. Strict XHTML Conformance: `03_intent_verification.html` must parse as valid XML
   (`xmlns="http://www.w3.org/1999/xhtml"`), with all tags self-closed or closed, attributes quoted, and ampersands
   escaped (`&amp;`).
6. No Deliberative History: The document must capture only the authoritative verification criteria, execution
   commands, and factual outcomes. This rule follows the repository principles. Deliberative debate belongs only in
   [`05_intent_conversation_summary.html`](intent-conversation-summary.md).

---

## 7. Revisions

| Date       | Version | Description                                                                                                                                                                                                                                                                                     | Source                |
|:-----------|:--------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------|
| 2026-11-25 | 1.2.1   | Rewrote the prose in plain English. No change to the design. | User request |
| 2026-11-25 | 1.2.0   | Evidence requirements and invariants now come from the solution strategy. Acceptance criteria come from the unit specifications. | TOPIC-007 |
| 2026-10-31 | 1.1.0   | Renamed to Intent Verification. Replaced the pre-execution stage with the feasibility stage, mapped evidence requirements and invariants to commands, added invariant monitors, gate types, and literal-output evidence. | TOPIC-005 |
| 2026-09-06 | 1.0.0   | Canonical specification for Intent Verification (`03_intent_verification.html`) establishing the dual-stage lifecycle, 1:1 requirements verification matrix, multi-level testing pyramid, non-functional quality gates, sub-intent/unit roll-up audits, manual sign-off protocols, and XPath query patterns. | Feature Specification |
| 2026-08-30 | 0.1.0   | Initial baseline stub in document catalog.                                                                                                                                                                                                                                                      | Initial Specification |