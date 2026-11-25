# Intent Specification Document Definition

The Intent Specification Document (`00_intent_spec.html`) is the entry-point document in the `brevio` work item
lifecycle. It records the intent in the originator's own words: the problem, the proposed outcome, the affected users and
systems, the constraints, and the open questions. It is a proto-spec. It holds no invariants, evidence requirements, or
design decisions. Those belong to the [Solution Strategy](solution-strategy.md) (`01_solution_strategy.html`).

The product owner accepts or closes the intent. After acceptance, the document is frozen, and the solution strategy pass
starts.

This document serves a dual purpose:

1. **Human Interface:** Renders as a local, fully interactive, static XHTML web page that uses Material Design 3 Web
   Components (`@material/web` / **M3**) without requiring any client-side bundlers or build steps.
2. **Agent / Harness Interface:** Serves as a deterministic, structured XML document queryable via XPath and the `brevio`
   CLI for token-efficient agent ingestion.

---

## Table of Contents

1. [File Location & Organization Standard](#1-file-location--organization-standard)
2. [XHTML Header & Metadata Standard](#2-xhtml-header--metadata-standard)
3. [Required Sections & M3 (Material Web) HTML Snippets](#3-required-sections--m3-material-web-html-snippets)
    - 3.1. [Document Header & Metadata Bar](#31-document-header--metadata-bar)
    - 3.2. [Problem](#32-problem)
    - 3.3. [Proposed Outcome](#33-proposed-outcome)
    - 3.4. [Affected Users & Systems](#34-affected-users--systems)
    - 3.5. [Constraints](#35-constraints)
    - 3.6. [Open Questions](#36-open-questions)
    - 3.7. [Related Work Items & Origin](#37-related-work-items--origin)
4. [Authoring & Acceptance](#4-authoring--acceptance)
5. [Token Optimization & XPath Query Patterns](#5-token-optimization--xpath-query-patterns)
6. [Rules & Constraints](#6-rules--constraints)
7. [Revisions](#7-revisions)

---

## 1. File Location & Organization Standard

The intent specification resides at the root of the work item directory:

```text
.brevio/
└── work_items/
    └── {work-item-id}/
        └── 00_intent_spec.html
```

- **File Name:** `00_intent_spec.html`
- **Format:** Strict XHTML (`application/xhtml+xml` compliant, standard XML syntax).
- **Encoding:** UTF-8.
- **Runtime:** Fully static, runnable directly in any modern browser via local filesystem (`file://`) or served locally
  via the `brevio` CLI.

---

## 2. XHTML Header & Metadata Standard

Every `00_intent_spec.html` file begins with the universal baseline `<meta>` tags, intent-specific tags, and the Material
Web ES module importmap loader. The example shows an accepted intent:

```xhtml

<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Intent: User Authentication Service</title>

    <!-- Universal Baseline Metadata -->
    <meta name="doc-id" content="WI-20260830T170936Z:00:SPEC"/>
    <meta name="doc-type" content="intent-spec"/>
    <meta name="schema-version" content="1.0.0"/>
    <meta name="domain" content="work-items"/>
    <meta name="intent-id" content="20260830T170936Z_user_auth_service"/>
    <meta name="status" content="completed"/> <!-- pending | in-progress | completed | failed | blocked -->
    <meta name="created-at" content="2026-08-30T17:10:00Z"/>
    <meta name="updated-at" content="2026-08-30T17:14:22Z"/>

    <!-- Intent Domain Metadata -->
    <meta name="convergence-status" content="formulating"/> <!-- declared | closed | formulating | converging | verified | settled -->
    <meta name="author" content="human:j.ortiz"/> <!-- human:{name} | agent:{name}@{version} -->
    <meta name="priority" content="high"/> <!-- low | medium | high | critical -->
    <meta name="business-impact" content="security"/> <!-- security | infrastructure | core-feature | bug-fix -->
    <meta name="risk-class" content="high"/> <!-- routine | high -->
    <meta name="accepted-by" content="human:product-owner"/> <!-- empty until accepted -->
    <meta name="accepted-at" content="2026-08-30T17:14:22Z"/> <!-- empty until accepted -->
    <meta name="question-count" content="2"/>
    <meta name="origin-ref" content=""/> <!-- finding or alert that created this intent -->
    <meta name="external-ref" content=""/> <!-- ID of a record in a legacy tracking system -->

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

---

## 3. Required Sections & M3 (Material Web) HTML Snippets

The Intent Specification contains 7 standardized sections. Each section uses semantic XHTML tags combined with Material
Web components (`md-*`) and typography tokens (`md-typescale-*`). The originator writes sections 3.2 to 3.6 in their own
words. No formal language is required.

### 3.1. Document Header & Metadata Bar

Anchors the document with the work item identifier, intent title, author, draft status, convergence status chip, risk
class, priority, and the acceptance record.

### 3.2. Problem

Section id: `problem`. States what the originator or the users cannot do today, and why it matters.

### 3.3. Proposed Outcome

Section id: `proposed-outcome`. States what better looks like. The outcome names a result, not an implementation.

### 3.4. Affected Users & Systems

Section id: `affected-users-systems`. Lists the users, teams, and systems that the intent affects.

### 3.5. Constraints

Section id: `constraints`. Lists the limits that the originator sets, for example "No new PII in the portal session" or
"Existing authentication only". Each constraint is an `md-list-item` with a `data-constraint-id` attribute (for example
`CON-001`). Items that are out of scope carry `data-scope="out"`. The solution strategy turns these constraints into
invariants and a boundary envelope.

### 3.6. Open Questions

Section id: `open-questions`. Each question is an `md-list-item` with a `data-question-id` attribute (for example
`Q-001`) and an owner. The solution strategy answers each question or carries it forward in its Open Question Resolution
section. This document does not record the answers.

### 3.7. Related Work Items & Origin

Section id: `related-items`. Lists upstream prerequisites and downstream dependents. For an intent that began from a
finding or an alert, it shows the `origin-ref` value. For a legacy tracking record, it shows the `external-ref` value.

---

## 4. Authoring & Acceptance

1. The originator describes the problem to an agent in their own words.
2. The agent asks the questions that an analyst asks: scope, users, constraints, and what success looks like.
3. The agent writes `00_intent_spec.html`. The originator corrects anything that the agent misunderstood.
4. The originator sets `risk-class`. The value is `routine` or `high`.
5. The product owner reviews the intent. If `risk-class` is `high`, a tech lead reviews it too.
6. If the product owner accepts the intent, `brevio` records `accepted-by` and `accepted-at`, and the intent moves to
   `formulating`.
7. If the product owner turns the intent down, the intent moves to `closed`.

An intent that began from a finding or an alert follows the same steps. An agent writes the intent, and the product owner
reviews and corrects it before acceptance.

---

## 5. Token Optimization & XPath Query Patterns

Agents operating in `brevio` query specific subsections using standard XPath 1.0/2.0 without loading entire documents,
preserving LLM context and minimizing costs.

| Extraction Objective      | XPath Query Pattern                                            |
|:--------------------------|:---------------------------------------------------------------|
| Problem statement         | `//section[@id='problem']//p`                                  |
| Proposed outcome          | `//section[@id='proposed-outcome']//p`                         |
| Constraints               | `//section[@id='constraints']//md-list-item`                   |
| Out-of-scope items        | `//section[@id='constraints']//*[@data-scope='out']`           |
| Open questions            | `//section[@id='open-questions']//md-list-item`                |
| Acceptance record         | `/html/head/meta[@name='accepted-by' or @name='accepted-at']/@content` |
| Risk class                | `/html/head/meta[@name='risk-class']/@content`                 |

---

## 6. Rules & Constraints

1. The originator's words come first. The agent records the intent and does not add design content.
2. The document must not contain invariants, evidence requirements, a boundary envelope, an autonomy tier, user stories,
   story points, or velocity metrics.
3. After acceptance, the document is frozen. A change after acceptance needs a new acceptance record, which updates
   `accepted-by` and `accepted-at`.
4. Every open question must have a `data-question-id` and an owner.
5. A `closed` intent is terminal. To restart the idea, the originator declares a new intent.
6. Revision history is tracked exclusively in the `## Revisions` table at the bottom of this document.

---

## 7. Revisions

| Date       | Version | Description                                                                                                                                                                                       | Source                           |
|:-----------|:--------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------|
| 2026-11-25 | 0.5.0   | Reduced the intent specification to the originator's proto-spec: separate problem, proposed outcome, affected users and systems, constraints, and open question sections. Added the acceptance record, `risk-class`, and the `closed` outcome. Moved invariants, evidence requirements, the boundary envelope, and the autonomy tier to the solution strategy. | TOPIC-007                        |
| 2026-10-31 | 0.4.0   | Replaced the requirements document with the intent specification: declared intent, system invariants with enforcement levels, boundary envelope, evidence requirements, autonomy tier, and origin tags. | TOPIC-005                        |
| 2026-09-05 | 0.3.0   | Removed the XHTML snippets as the UI defined ones generated in the initial draft were junk.                                                                                                       | [Work Item Documents](README.md) |
| 2026-09-05 | 0.2.0   | Standardized RFC 2119 levels, User Story framing, complexity/points estimation, dedicated Acceptance Criteria checklists with `data-req-ref`, and status-chipped Q&amp;A cards.                   | [Work Item Documents](README.md) |
| 2026-09-05 | 0.1.0   | Initial baseline requirements specification with Material Web (M3) snippets.                                                                                                                      | [Work Item Documents](README.md) |
