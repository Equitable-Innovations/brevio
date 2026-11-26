# TOPIC-###: [Topic Title]

| Metadata | Value |
| :--- | :--- |
| **Topic ID** | `TOPIC-###` |
| **Status** | `PROPOSED` | <!-- Options: PROPOSED | IN_DISCUSSION | DECIDED | GRADUATED | DEFERRED | REJECTED -->
| **Facilitator** | @username |
| **Participants** | @username1, @username2 |
| **Created Date** | YYYY-MM-DD |
| **Last Updated** | YYYY-MM-DD |
| **Target Build-Docs** | `build-docs/...` |
| **Impacted Domains** | `CLI` \| `MCP` \| `Static UI` \| `Work Items` \| `Wiki` \| `OpenAPI` |

---

## 1. Problem Statement & Context

### 1.1 Problem Description
What specific problem, feature set, or architectural gap does this topic solve?

### 1.2 Context & Trigger
Why are we discussing this now during the product ideation stage? What background or constraints are relevant?

---

## 2. Goals & Non-Goals

### Goals
- [ ] Goal 1: ...
- [ ] Goal 2: ...

### Non-Goals
- Deliberately out of scope item 1.
- Deliberately out of scope item 2.

---

## 3. Token Optimization & Performance Impact
*`brevio` uses local processing to use as few LLM tokens as possible and to make development as fast as possible.*

- **Context Window & Token Economics**: How does this design affect input/output tokens? Does it avoid the load of documentation that is not necessary?
- **Speed & Latency**: Does this make local parsing (for example, XPath / XHTML queries) or fast offline command execution possible?

---

## 4. Proposed Options & Trade-Off Analysis

### Option A: [Option Name]
- **Summary**: Concise description of this approach.
- **Pros**:
  - Point 1
  - Point 2
- **Cons**:
  - Point 1
  - Point 2
- **Token / Latency Overhead**: Analysis of costs.
- **Complexity**: Low / Medium / High.

### Option B: [Option Name]
- **Summary**: Concise description of this approach.
- **Pros**:
  - Point 1
  - Point 2
- **Cons**:
  - Point 1
  - Point 2
- **Token / Latency Overhead**: Analysis of costs.
- **Complexity**: Low / Medium / High.

---

## 5. Open Questions & Spikes

- [ ] **Q1**: Description of question / unknown. *(Owner: @user)*
- [ ] **Q2**: Description of question / spike task. *(Owner: @user)*

---

## 6. Discussion Log & Meeting Notes

- **YYYY-MM-DD** (@user): Initial topic proposal created.
- **YYYY-MM-DD** (@user1, @user2): Notes on architectural review, trade-off feedback, or team debate.

---

## 7. Decision & Rationale
*(When you change the status to `DECIDED`, complete this section.)*

- **Decision**: Selected approach (for example, Option A, Option B, or Hybrid).
- **Core Rationale**: Why did the team choose this over the alternatives?
- **Accepted Trade-offs**: What limits or added complexity did the team knowingly accept?
- **Decision Date**: YYYY-MM-DD
- **Sign-off**: @stakeholder1, @stakeholder2

---

## 8. Build-Docs Conversion Plan
*(Complete this section during the transition from `DECIDED` to `GRADUATED`.)*

### 8.1 Target Build-Docs Destinations
- [ ] Destination: `build-docs/...` (for example, `build-docs/cli/README.md`)

### 8.2 Specification Authoring Checklist
- [ ] Drafted normative, prescriptive technical specifications.
- [ ] Made sure that the content follows the rules in [`build-docs/README.md`](../build-docs/README.md):
  - [ ] Content shows the current state of the design.
  - [ ] Content contains no references to past decisions, debate, or rejected alternatives (those stay here in this topic).
- [ ] Added an entry to the `## Revisions` table in the target build-doc. The entry references this `TOPIC-###`.
- [ ] Updated status in this topic to `GRADUATED`.
- [ ] Updated Topic Registry table in [`topic-registry.md`](topic-registry.md).
