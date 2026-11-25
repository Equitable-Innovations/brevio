# TOPIC-007: Work Item Alignment with the SDLC Playbook Intent, Spec, and Plan

| Metadata | Value |
| :--- | :--- |
| **Topic ID** | `TOPIC-007` |
| **Status** | `GRADUATED` | <!-- Options: PROPOSED | IN_DISCUSSION | DECIDED | GRADUATED | DEFERRED | REJECTED -->
| **Facilitator** | @samsassin |
| **Participants** | @samsassin, Claude |
| **Created Date** | 2026-11-25 |
| **Last Updated** | 2026-11-25 |
| **Target Build-Docs** | `build-docs/docs/work-items/`, `build-docs/docs/README.md`, `build-docs/project-mgr/wts/README.md`, `build-docs/static-ui/README.md` |
| **Impacted Domains** | `Work Items` \| `Static UI` |

---

## 1. Problem Statement & Context

### 1.1 Problem Description

[TOPIC-005](TOPIC-005_intent-based-sdlc-work-items.md) mapped the Work Items domain to the stage model of the AI-native
SDLC playbook (`AI Native SDLC Playbook_designv2.pdf`). The stage mapping holds. A second review of the first three
documents found gaps in what each document holds and in who accepts it.

The playbook uses three committed artifacts before any code changes:

| Playbook artifact | Contents | Accepted by | Starts |
| :--- | :--- | :--- | :--- |
| `intent.md` | Author and status, problem, proposed outcome, affected users and systems, constraints, open questions. Written in the originator's words. | Product owner | The design pass |
| `spec.md` | A requirements and design spec, written by an agent under the organization's skills, with flagged concerns. Answers or carries forward each open question. | Product owner, with a tech lead for higher-risk work | Plan mode |
| `plan.md` | Files that change, order of work, risks, and proof. Interrogated for what can break, the riskiest step, and the options not chosen. | Engineer, or a tech lead or architect for higher-risk work | The build |

The current `brevio` documents differ from these artifacts in six ways:

1. **The accepted intent changes after acceptance.** `00_intent_spec.html` holds the originator's proto-spec and also the
   invariants, evidence requirements, boundary envelope, and autonomy tier. The agent adds these during `formulating`,
   after the product owner accepts the intent. The playbook metric "edits to the intent after the first spec commit"
   then counts every formulation edit as rework.
2. **The intent sections are merged.** Section `declared-intent` holds the problem, the outcome, the affected users, and
   the constraints in one section. An agent cannot query one part alone. The boundary envelope overlaps the constraints.
3. **No state records a closed intent.** The lifecycle has no state for an intent that the product owner turns down.
   `brevio` cannot report the playbook's survival rate.
4. **The solution strategy mixes spec content and plan content.** `01_solution_strategy.html` asks for class and symbol
   names, file paths, DDL snippets, model tiers, and workspace modes. The playbook puts this detail in `plan.md`. A
   product owner without engineering skill cannot review it. The strategy has no requirements section, and the open
   question carry-forward has no section of its own. The header has `policy-versions` but no `prompt-ref`.
5. **Flagged concerns are resolved too late.** The playbook resolves each concern before engineering sees the spec.
   `brevio` blocks only the move to `converging`, so the decomposition can be written while a concern is open.
6. **The plan has no single reviewable view.** The order is in the `02` DAG and the risks are in the `02` catalog. The
   files are spread across the Target Scope of each unit spec, and the proof is spread across each verification contract.
   `02` has no interrogation section and no "implementable from the plan alone" test. The playbook metric "how often the
   merged diff still matches the committed plan" needs one plan-level file list.

The review also found these smaller issues:

- No document ties the approver role to a risk class. The playbook adds a tech lead or architect for higher-risk work.
- README rule 1 bans rejected alternatives in generated documents. Micro-ADRs list trade-offs, and the playbook asks the
  plan to record the options not chosen.
- Every intent must write the full three-tier hierarchy. The playbook plans a small change in one `plan.md`.
- The `00` header example shows `status=completed` next to `convergence-status=declared`.
- Unit spec section 3.8 lists acceptance criteria that are not commands, for example "Migration file exists". This
  conflicts with rule 4 of the same document.

### 1.2 Context & Trigger

The user asked for a review of `build-docs/docs/work-items/` against the playbook design for intent, design spec, and
plan. `brevio` implementation started with WORK-ITEM-001. The document structure must be right before the
`brevio work-item` commands generate these documents.

---

## 2. Goals & Non-Goals

### Goals

- [x] After the product owner accepts `00_intent_spec.html`, freeze it.
- [x] Make `01_solution_strategy.html` a requirements and design spec that a product owner can review.
- [x] Give `02_intent_decomposition.html` a one-view plan that a reviewer can accept in one read.
- [x] Resolve flagged concerns before the decomposition pass starts.
- [x] Tie each acceptance to an approver role chosen by a risk class.
- [x] Let a small intent skip the sub-intent tier.

### Non-Goals

- Rename the documents to `intent`, `spec`, and `plan`. TOPIC-005 set the current names.
- Merge `00` and `01` into one document. TOPIC-005 Q4 kept two documents.
- Change the convergence loop, the ledger scheduling rules, or the unit verification documents beyond the moved
  evidence source.
- Switch the document format from XHTML to Markdown.

---

## 3. Token Optimization & Performance Impact

- **Context Window & Token Economics**: Separate section ids in `00` let an agent read only the problem or only the
  constraints. The plan view in `02` gives a reviewer agent one sub-tree to read and not every unit spec. The light path
  removes three sub-intent documents from a one-milestone intent. Moving the engineering detail from `01` to `02` does
  not add tokens. It moves them to the document that the build agent reads.
- **Speed & Latency**: The CLI computes the plan-level file list as the union of the unit Target Scopes. Plan drift is
  then one local comparison between the merged diff and that list.

---

## 4. Proposed Options & Trade-Off Analysis

### Decision Area 1: Where invariants, evidence, and boundaries live

#### Option 1A: Move them to the solution strategy

- **Summary**: `00` holds only the originator's proto-spec and is frozen at acceptance. Invariants, evidence
  requirements, the boundary envelope, and the autonomy tier move to `01`.
- **Pros**:
  - The accepted intent never changes, so intent rework is measurable.
  - `01` becomes the requirements and design spec that the playbook describes.
- **Cons**:
  - Every document that cites `00` for invariants or evidence changes its citation to `01`.
- **Token / Latency Overhead**: None.
- **Complexity**: Medium.

#### Option 1B: Keep them in `00` with split acceptance

- **Summary**: `00` keeps both parts. The originator part has its own hash and acceptance record. The formulation part
  is accepted later with `01`.
- **Pros**:
  - Fewer citation changes.
- **Cons**:
  - One document has two acceptance records and two hashes.
  - Rework metrics must filter by section.
- **Token / Latency Overhead**: None.
- **Complexity**: Medium.

#### Option 1C: No change

- **Summary**: The intent document changes during `formulating`.
- **Pros**:
  - No work.
- **Cons**:
  - `brevio` cannot adopt the playbook intent-rework metric.
- **Token / Latency Overhead**: None.
- **Complexity**: Low.

### Decision Area 2: How `01` and `02` split spec and plan content

#### Option 2A: Move the detail to `02` and add a plan view

- **Summary**: File paths, symbol names, DDL, and agent model and tooling configuration move from `01` to `02`. `02`
  adds a plan view (files that change, order, risks, proof) and an interrogation section.
- **Pros**:
  - A product owner can review `01`.
  - One section of `02` is the reviewable plan, and drift is measured against it.
- **Cons**:
  - The largest edit of the three options.
- **Token / Latency Overhead**: Neutral. The content moves.
- **Complexity**: Medium.

#### Option 2B: Add the plan view only

- **Summary**: `01` stays as it is. `02` adds the plan view.
- **Pros**:
  - Smaller edit.
- **Cons**:
  - `01` stays unreadable for a product owner.
- **Token / Latency Overhead**: Low.
- **Complexity**: Low.

#### Option 2C: Slim `01` only

- **Summary**: The detail moves out of `01`. No plan view is added.
- **Pros**:
  - `01` becomes reviewable.
- **Cons**:
  - The plan stays spread across many documents.
- **Token / Latency Overhead**: None.
- **Complexity**: Low.

### Decision Area 3: Small intents

#### Option 3A: A light path

- **Summary**: An intent with one milestone uses one implicit sub-intent. `02` lists its units directly. No sub-intent
  spec, verification, or summary files are written.
- **Pros**:
  - Matches the playbook's single `plan.md` for a small change.
  - Fewer documents and fewer tokens.
- **Cons**:
  - Tools must handle two layouts.
- **Token / Latency Overhead**: Lower for small intents.
- **Complexity**: Medium.

#### Option 3B: Always the full hierarchy

- **Summary**: Every intent writes `02`, at least one sub-intent folder, and unit folders.
- **Pros**:
  - One layout for tools.
- **Cons**:
  - Overhead for small changes.
- **Token / Latency Overhead**: Higher for small intents.
- **Complexity**: Low.

#### Option 3C: Defer

- **Summary**: Record the question and keep the current hierarchy.
- **Pros**:
  - No work now.
- **Cons**:
  - After the CLI starts to generate documents, the question returns.
- **Token / Latency Overhead**: None now.
- **Complexity**: Low.

### Decision Area 4: Alternatives in generated documents

#### Option 4A: Allow alternatives in `01` and `02`

- **Summary**: README rule 1 governs `build-docs/` and the current-state sections of generated documents. Micro-ADRs in
  `01` and the interrogation section in `02` can record the options not chosen.
- **Pros**:
  - Matches the playbook plan interrogation.
  - Removes the conflict with micro-ADR trade-offs.
- **Cons**:
  - The rule has two scopes.
- **Token / Latency Overhead**: Small. An agent reads these sections only on request.
- **Complexity**: Low.

#### Option 4B: Keep rule 1 strict

- **Summary**: Generated documents hold only the current decision. Alternatives stay in `05_intent_conversation_summary.html`.
- **Pros**:
  - One rule for every document.
- **Cons**:
  - The reviewer of the plan cannot see the options not chosen in the plan.
- **Token / Latency Overhead**: None.
- **Complexity**: Low.

---

## 5. Open Questions & Spikes

- [x] **Q1**: Where do invariants, evidence requirements, the boundary envelope, and the autonomy tier live? *(Owner: @samsassin)*
  - **Resolved 2026-11-25**: Option 1A. They move to `01_solution_strategy.html`. `00_intent_spec.html` is frozen at acceptance.
- [x] **Q2**: How do `01` and `02` split spec and plan content? *(Owner: @samsassin)*
  - **Resolved 2026-11-25**: Option 2A. The engineering detail moves to `02`, and `02` adds a plan view and an interrogation section.
- [x] **Q3**: Can a small intent skip the sub-intent tier? *(Owner: @samsassin)*
  - **Resolved 2026-11-25**: Option 3A. A one-milestone intent uses one implicit sub-intent, and `02` lists its units directly.
- [x] **Q4**: Can generated documents record alternatives not chosen? *(Owner: @samsassin)*
  - **Resolved 2026-11-25**: Option 4A. Micro-ADRs in `01` and the interrogation section in `02` can record them.

---

## 6. Discussion Log & Meeting Notes

- **2026-11-25** (Claude): Reviewed `build-docs/docs/work-items/` against the PDF edition of the playbook. Found the six
  gaps and the smaller issues in Section 1.1. Proposed the four decision areas in Section 4.
- **2026-11-25** (@samsassin): Chose Option 1A, Option 2A, Option 3A, and Option 4A. Asked to record the decision in this
  topic before the build-docs change.
- **2026-11-25** (Claude): Graduated the decision into the build-docs. Seven more documents cited `00_intent_spec.html`
  for evidence, invariants, or the autonomy tier: the intent summary, conversation summary, conversation metrics, review
  policy, docs index, WTS mapping, and static UI templates. They now cite the solution strategy. The sub-intent spec
  layout also used a stale `phase_{NN}` directory name, which is now `sub_intent_{NN}`.

---

## 7. Decision & Rationale

- **Decision**: Options 1A, 2A, 3A, and 4A, with these supporting changes:
  - **Intent (`00`)**: Separate section ids for `problem`, `proposed-outcome`, `affected-users-systems`, `constraints`,
    `open-questions`, and `related-items`. Add `accepted-by` and `accepted-at`. After the product owner accepts the
    document, it is frozen. A change after acceptance needs a new acceptance record.
  - **Lifecycle**: Add a `closed` state for an intent that the product owner turns down.
  - **Strategy (`01`)**: Add a requirements section, a separate open-question carry-forward section keyed to the question
    IDs in `00`, and a `prompt-ref` tag. Take the invariants, evidence requirements, boundary envelope, and autonomy tier
    from `00`. Keep the flagged concerns and the micro-ADRs. The product owner cannot accept `01` while any concern is
    `open`.
  - **Decomposition (`02`)**: Take the component and module changes, data model and schema changes, and agent tooling
    configuration from `01`. Add a plan view: files that change (the union of the unit Target Scopes), order, risks, and
    proof. Add an interrogation section: what the change can break, the riskiest step, and the options not chosen. Add
    an adversarial gate that tests whether an engineer who never saw the conversation can implement the change from the
    plan alone.
  - **Risk class**: Add a `risk-class` tag (`routine`, `high`). The product owner accepts `00` and `01`, with a tech lead
    for `high`. An engineer accepts `02`, or a tech lead or architect for `high`.
  - **Light path**: A one-milestone intent uses one implicit sub-intent. Its units sit under `units/` at the work item
    root. The ledger keeps one sub-intent entry, so scheduling does not change.
  - **Rule 1**: README rule 1 governs `build-docs/` and the current-state sections of generated documents.
  - **Fixes**: Correct the `00` status example. Make each unit acceptance criterion a command with a pass or fail result.
- **Core Rationale**: The playbook gives each stage one accepted artifact, and the next stage reads it. A document that
  changes after acceptance breaks the audit trail and the git-derived metrics. Each reviewer reads the content that they
  can judge: the product owner reads the problem and the requirements, and the engineer reads the files and the order.
- **Accepted Trade-offs**:
  - Every document that cites `00` for invariants or evidence must cite `01`.
  - Tools must handle two layouts: the full hierarchy and the light path.
  - README rule 1 has two scopes.
- **Decision Date**: 2026-11-25
- **Sign-off**: @samsassin

---

## 8. Build-Docs Conversion Plan

### 8.1 Target Build-Docs Destinations

- [x] Destination: `build-docs/docs/work-items/README.md` (hierarchy, state table, gates, risk class approvers,
  catalog, light path layout, metadata matrix, handoffs, traceability, rules)
- [x] Destination: `build-docs/docs/work-items/intent-specification.md`
- [x] Destination: `build-docs/docs/work-items/solution-strategy.md`
- [x] Destination: `build-docs/docs/work-items/intent-decomposition.md`
- [x] Destination: `build-docs/docs/work-items/sub-intent-specification.md` (light path)
- [x] Destination: `build-docs/docs/work-items/intent-unit-specification.md` (light path location, acceptance criteria)
- [x] Destination: `build-docs/docs/work-items/intent-verification.md` (evidence source)
- [x] Destination: `build-docs/docs/work-items/intent-ledger.md` (`closed` state, light layout, planning gates)
- [x] Destination: `build-docs/docs/work-items/intent-document-metadata-definition.md` (`accepted-by`, `accepted-at`,
  `risk-class`, `prompt-ref`, `layout`, `plan-files`)
- [x] Destination: `build-docs/docs/work-items/intent-summary.md` (evidence source)
- [x] Destination: `build-docs/docs/work-items/intent-conversation-summary.md` (document roles)
- [x] Destination: `build-docs/docs/work-items/intent-conversation-metrics.md` (evidence source)
- [x] Destination: `build-docs/docs/work-items/review-policy.md` (compliance pass sources)
- [x] Destination: `build-docs/docs/README.md` (Work Items artifact list)
- [x] Destination: `build-docs/project-mgr/wts/README.md` (intent package mapping)
- [x] Destination: `build-docs/static-ui/README.md` (planning document templates)

### 8.2 Specification Authoring Checklist

- [x] Drafted normative, prescriptive technical specifications.
- [x] Verified compliance with [`build-docs/README.md`](../build-docs/README.md) rules:
  - [x] Content represents the **current state** of the design.
  - [x] Content is **strictly void** of references to past decisions, debate, or rejected alternatives (those remain here in this topic).
- [x] Added entry to the `## Revisions` table in the target build-doc referencing this `TOPIC-007`.
- [x] Updated status in this topic to `GRADUATED`.
- [x] Updated Topic Registry table in [`topic-registry.md`](topic-registry.md).
