# Brevio Build Order

This document defines the high-level build order for `brevio`. The order reaches a usable minimum viable product (MVP) as fast as possible. Then it adds the remaining domains in steps. Each phase is the starting point for its detailed work items.

---

## Sequencing Strategy

- Build the shared primitives once. Before you build any domain command, finish atomic file I/O, locking, strict XHTML parsing and rendering, and document validation.
- Ship a self-hosting MVP first (Phases 1 to 4). Self-hosting means that `brevio` manages its own development. Deliver `doc`, `wts`, `work-item`, the core `mcp` server, and the agent assets early. With these, `brevio` can run end-to-end intent-driven workflows before work on the secondary domains starts.
- Build the post-MVP domains in parallel (Phases 5 to 7). Project Manager, Project Wiki, and OpenAPI all use the same core document engine and MCP transport. After the MVP, you can build them in parallel, or ship each one on its own in priority order.
- Deliver each domain as a vertical slice. A slice holds the CLI commands, the MCP tools, and the Static UI templates of the domain. Do not leave MCP or UI rendering to the end of the project.

---

## Build Order

- Phase 0: CLI Skeleton & Workspace Bootstrap (Complete: `WORK-ITEM-001`)
  - This phase built the Go module, the Cobra command tree, and the global flags (`--root`, `--format`, `--quiet`, `--help`). It also built the exit codes (`0` to `6`), the JSON and text output printer, the CI workflow, and `brevio init`.
  - Specification: [`cli/README.md`](cli/README.md)

- Phase 1: Core Document Engine, Storage Primitives & `brevio doc`
  - Build workspace root discovery, which finds the nearest parent `.brevio/`. Build the loader for `.brevio/brevio.config.json`.
  - Build shared helpers for atomic multi-file writes. Each helper writes a temporary file and then renames it. Build the file-locking and optimistic concurrency primitives. A conflict exits with code `6`.
  - Build the universal strict XHTML parser and serializer (`xmlns="http://www.w3.org/1999/xhtml"`). Build the reader and writer for the 7 universal baseline `<meta>` tags. Build the zero-build Material Design 3 (M3) base HTML shell.
  - Build the document schema registry. When you write each schema, reconcile the open metadata tag definitions.
  - Build the `brevio doc` commands: `validate`, `query` (`--xpath` for XHTML, `--jq` for JSON and JSONL), and `schema`.
  - Specifications: [`docs/README.md`](docs/README.md), [`document-types.md`](document-types.md), [`metadata-elements.md`](metadata-elements.md), [`static-ui/README.md`](static-ui/README.md), [`cli/README.md`](cli/README.md)

- Phase 2: Work Tracking System (`brevio wts`) (Self-Hosting Milestone)
  - Build the backlog index store (`.brevio/wts/backlog.jsonl`), the Markdown templates for work items, and the deterministic `backlog.md` generator.
  - Build the lifecycle transition rules, the date stamps (`created`, `active`, `updated`, `completed`), and the status gates. `READY` requires invariant and evidence commands. `DONE` requires passing results.
  - Build the `brevio wts` commands: `next-id`, `add`, `edit`, `remove`, `generate`, `list`, and `check`. The `check` command covers the core drift checks between the files, the index, and the view.
  - `brevio wts` replaces the bootstrap backlog script, so `brevio` tracks its own remaining build work.
  - Specification: [`project-mgr/wts/README.md`](project-mgr/wts/README.md)

- Phase 3: Work Items Domain & Intent Convergence Engine (`brevio work-item`)
  - Build runtime intent package seeding (`brevio wts start`). It creates `.brevio/work_items/{timestamp}_{slug}/`. It maps the WTS sections into the Tier 1, Tier 2, and Tier 3 XHTML documents and into the project-level `review_policy.html` and `lessons.html`.
  - Build the state machine of the Intent Ledger (`03_intent_ledger.json`). It uses `.ledger.lock` and `ledgerVersion` for optimistic concurrency. It holds the convergence states, the guards for dependencies and disjoint scopes, the retry budgets and `retry_budget_exceeded` holds, and the separation-of-duties checks.
  - Build the `brevio work-item` commands: `ledger slice`, `unit start`, `unit verify`, `unit complete`, and `hooks install`. The `hooks install` command writes the `build-gate` hook templates for protected paths and locked test files.
  - Build the Work Items M3 Static UI templates: intent, strategy, decomposition and ledger, sub-intent, unit, review findings, lessons, and the conversation and metrics views.
  - Specifications: [`docs/work-items/README.md`](docs/work-items/README.md), [`docs/work-items/intent-ledger.md`](docs/work-items/intent-ledger.md), [`static-ui/README.md`](static-ui/README.md)

- Phase 4: MCP Server Core & Agent Assets (MVP Launch Cut)
  - Build the `brevio mcp` stdio server. It shares the internal Go service packages of the CLI. It returns stable error codes (`not_found`, `invalid_input`, `invalid_document`, `gate_failed`, `conflict`, `forbidden`). It applies the human-only authority limits.
  - Build the MVP MCP tools. The Document tools are `brevio_query_doc`, `brevio_get_schema`, and `brevio_validate_doc`. The Work Item tools are `brevio_ledger_slice`, `brevio_update_unit`, and `brevio_get_gate_state`. The WTS tools are `brevio_wts_list`, `brevio_wts_add`, `brevio_wts_edit`, `brevio_wts_start`, and `brevio_wts_check`.
  - Build the MVP MCP resource `brevio://wts/backlog`.
  - Build `brevio assets install`. It bundles and installs the WTS skills (`create-work-item`, `update-work-item-status`, `update-backlog`) and the `backlog-manager` agent.
  - Exit checkpoint: developers and LLM agents can use `brevio` end-to-end for token-optimized, intent-driven delivery.
  - Specifications: [`mcp/README.md`](mcp/README.md), [`cli/README.md`](cli/README.md), [`project-mgr/wts/README.md`](project-mgr/wts/README.md)

- Phase 5: Project Manager (`brevio pm`) (Post-MVP, can run in parallel with Phases 6 and 7)
  - Build document scaffolding and management: `brevio pm init` (`prd.html`, `fsd.html`), `initiative add`, `epic add`, and `milestone add`. These documents have parent and child rollup lifecycles.
  - Build the dependency DAG (directed acyclic graph) in `.brevio/project-mgr/edges.jsonl`. Build these commands for it: `edge add` (DFS cycle detection), `edge remove`, and `graph` (Kahn's topological sort and `--reduce` transitive reduction).
  - Build traceability and governance: `pm trace` (generates `traceability.html`), `pm gate`, and `pm check`. The `pm gate` command runs the Completion, Convergence, Findings, and Coverage gates and the human-only Authorization gate.
  - Connect the domains. Connect WTS `check` to `edges.jsonl` and to the existence of epics. Propagate ledger holds to the WTS `BLOCKED` status and to the milestone Convergence gates.
  - Build the PM M3 Static UI templates, the MCP PM tools (`brevio_pm_*`), and the resources (`brevio://pm/traceability`, `brevio://pm/milestone/{version}`).
  - Specifications: [`project-mgr/README.md`](project-mgr/README.md), [`mcp/README.md`](mcp/README.md), [`static-ui/README.md`](static-ui/README.md)

- Phase 6: Project Wiki (`brevio wiki`) (Post-MVP, can run in parallel with Phases 5 and 7)
  - Build wiki bootstrap and indexing: `brevio wiki init` (5 core pages, `index.json`, and `index.html`), `wiki index`, and `wiki check`. The `wiki check` command covers schema validation, broken links, index drift, and git-based `review_due` detection.
  - Build the read and query commands that use few tokens: `list`, `outline`, `get`, `find`, and `rules`. The `rules` command filters the coding standards by path glob.
  - Build section-level authoring and governance: `page add/set-meta/move/remove`, `section set/add/remove` (optimistic locking with `--expect-hash`), `adr add/accept/deprecate`, `risk add/set`, and `review`, with enforcement of the human-only actions.
  - Build the Wiki M3 Static UI templates: a client-side navigation drawer built from `index.json`, draft and ADR chips, and review-due and superseded banners. Build the MCP Wiki tools (`brevio_wiki_*`) and the `brevio://wiki/index` resource.
  - Specifications: [`docs/wiki/README.md`](docs/wiki/README.md), [`mcp/README.md`](mcp/README.md), [`static-ui/README.md`](static-ui/README.md)

- Phase 7: OpenAPI Domain (`brevio open-api`) (Post-MVP, can run in parallel with Phases 5 and 6)
  - Build a local OpenAPI 3.0 and 3.1 parser and validator. It supports relative `$ref` values inside `source/`. It rejects remote URLs and makes zero network calls.
  - Build the deterministic XHTML projection engine. It generates `catalog.json`, `catalog.html`, the API overviews, one document for each operation, and one document for each schema. It tracks the SHA-256 `source-hash` and manages the stale and invalid states.
  - Build the `brevio open-api` commands: `add`, `update`, `generate`, `check`, `list`, `operation`, `schema`, and `remove`.
  - Build the OpenAPI M3 Static UI templates: endpoint cards, HTTP method chips, parameter and schema tables, and response accordions. Build the read-only MCP OpenAPI tools (`brevio_openapi_*`) and the `brevio://open-api/catalog` resource.
  - Specifications: [`docs/open-api/README.md`](docs/open-api/README.md), [`mcp/README.md`](mcp/README.md), [`static-ui/README.md`](static-ui/README.md)

- Phase 8: Unified Static UI Polish, Cross-Domain Integrity & Release Hardening
  - Build single-pane Static UI navigation (a rail or a drawer) that connects all four domains: Work Items, Project Manager, Wiki, and OpenAPI. Add light and dark M3 themes. Make sure that the UI works offline through `file://`. Add interactive dialogs: a diff viewer, a transcript viewer for conversation JSONL, and a telemetry dashboard.
  - Build validation of the cross-domain graph (`references-wiki`, `references-api`, `implements-req`, `parent-doc-id`) across all domains.
  - Build an end-to-end integration test suite. It covers the CLI, MCP, concurrency and locking, gate enforcement, and browser rendering. Build cross-platform packaging for the binary releases.
  - Specifications: [`static-ui/README.md`](static-ui/README.md), [`docs/README.md`](docs/README.md), [`commands.md`](commands.md)

---

## Guideline Check

These guidelines from [`guidelines/README.md`](guidelines/README.md) apply:

- 1 (KISS), 2 (YAGNI), 42 (Simplicity first): The plan cuts an explicit MVP at Phase 4 (`doc`, `wts`, `work-item`, and the core `mcp`). The core value ships without a wait for the secondary domains.
- 5 (DRY), 7 (Separation of concerns), 8 (Low coupling): The plan builds the shared storage, locking, XHTML, and validation primitives in Phase 1. From Phase 4 onward, the CLI and MCP share internal service code.
- 24 (Fail fast), 25 (Validate input at the boundary): The plan puts strict XHTML and metadata validation (`brevio doc validate`) and atomic writes in Phase 1, before any domain writers exist.
- 35 (Make small changes), 38 (Automate repeated work): The plan puts `brevio wts` early so that `brevio` tracks its own work. It structures Phases 5 to 7 as independent vertical slices.
- 39 (Handle security from the start): The plan enforces the human-only authority limits, the protected-file `build-gate` hooks, and the zero-network local `$ref` rules inside each domain phase.
- Guidelines broken: None.

---

## Revisions

| Date | Version | Description | Source |
| :--- | :--- | :--- | :--- |
| 2026-11-25 | 0.1.0 | Initial high-level build order across all `build-docs/` subsystems with MVP cut and post-MVP parallel tracks. | User request |
| 2026-11-25 | 0.1.1 | Rewrote the prose in plain English. No change to the design. | User request |
