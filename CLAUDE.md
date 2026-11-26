# CLAUDE.md

This file gives guidance to Claude Code (claude.ai/code) for work with code in this repository.

## What this repository is

This repo holds the design and specification for `brevio` (Token Optimized Software Harness), a CLI-based agentic
development harness. It also holds the first Go code for `brevio`. The Go skeleton and the `brevio init` command are built
([WORK-ITEM-001](backlog/WORK-ITEM-001_go-project-skeleton-and-brevio-init-command.md), `DONE`). Every other command is
still only a specification. Most of the repository is Markdown (and later XHTML). It specifies what `brevio` will do and
the structure of the documents that it will generate.

### Go code

- Module: `github.com/Equitable-Innovations/brevio`, Go 1.26, `cobra` for the command tree.
- Layout: `cmd/brevio/` is the entry point. `internal/cli/` holds the root command, the global flags (`--root`,
  `--format`, `--quiet`), exit codes, `ExitError`, the environment, and the printer. `internal/initcmd/` holds the
  `brevio init` logic.
- Commands (only these commands exist, so do not invent others):
  - `go build ./...`
  - `go vet ./...`
  - `go test ./...`
  - `go build -o bin/brevio ./cmd/brevio` (`bin/` is git-ignored)
- CI: `.github/workflows/ci.yml` runs build, vet, and test on every push and pull request. There is no lint or format
  command other than `go vet`.
- Behavior built so far: `brevio init` creates `.brevio/` and an empty `.brevio/brevio.config.json`. If `.brevio/` exists, it
  changes nothing, prints a message, and exits with code `2`.
- Add each new command group with real behavior, not as an empty stub. Follow the
  [CLI specification](build-docs/cli/README.md).

### Work tracking

- `backlog/` holds the development backlog. Each work item is `backlog/WORK-ITEM-###_<slug>.md` and has a record in
  `backlog/backlog.jsonl`.
- `backlog/backlog.md` is a generated file. Never edit it by hand. Use `node backlog/scripts/backlog.mjs`, or the
  `create-work-item`, `update-work-item-status`, and `update-backlog` skills.
- `build-work/` holds the working documents (intent spec, ledger, unit specs, verification) that a work item
  produces during its build.

The core idea of the specification: `brevio` will manage project documentation as strict XHTML files. Humans can read
these files in a browser, rendered with Material Design 3 web components. LLM agents can read them through XPath and XML
subtree extraction. Because agents do not read full documents, token use decreases.

## Repository Reference Documentation

- Material Design: [Material Design 3 (M3)](https://m3.material.io/)
- Web Component Online: [Material Web (`@material/web`)](https://material-web.dev/)
- Web Components Context7: Query `mcp__context7` with library id `/material-components/material-web`

## Repository structure & document pipeline

There are two top-level document domains. Content moves in one direction only, from `topics/` to `build-docs/`:

- `topics/`: the workspace for ideas and architectural decisions. Its content is exploratory and keeps the history,
  the debate, and the rejected alternatives. Each topic is a numbered file `topics/TOPIC-###_<kebab-case-slug>.md` (copy
  `topics/template.md`). Each topic goes through a lifecycle: `PROPOSED` → `IN_DISCUSSION` → `DECIDED`/`DEFERRED`/`REJECTED` →
  `GRADUATED`. `topics/topic-registry.md` indexes all topics. When you add a topic or a topic changes state, you must
  update `topics/topic-registry.md`.
- `build-docs/`: the canonical, prescriptive specification for the build of `brevio`. When a topic gets to `DECIDED`,
  copy its conclusions into `build-docs/`. Then mark the topic `GRADUATED`, with a link back to the build-doc (s) that it
  produced.

Critical rule for edits to `build-docs/`: the content there must show only the *current* state of the design. Never
write deliberative language ("we decided X over Y because...", comparisons, rejected alternatives) into
`build-docs/`. That history belongs only in the source `topics/` file. Every `build-docs/` document records its change
history only in a `## Revisions` table at the bottom. That table must cite the source `TOPIC-###` that produced the
change.

`build-docs/docs/README.md` is the master specification for the document architecture. It divides into three document
domains. Each domain has its own directory and README under `build-docs/docs/`. The table also lists the Project Manager
(`build-docs/project-mgr/`) and WTS, which is inside the Project Manager at `build-docs/project-mgr/wts`:

| Domain         | Directory                     | Status                                                                                   |
|----------------|-------------------------------|------------------------------------------------------------------------------------------|
| Work Items     | `build-docs/docs/work-items/` | Complete (18 specs). It is the reference example for depth and format.                   |
| Project Wiki   | `build-docs/docs/wiki/`       | Specified (README) ([TOPIC-002](topics/TOPIC-002_wiki-documents-domain-definition.md)): core pages, ADRs, freeform pages, section-level reads and writes |
| OpenAPI        | `build-docs/docs/open-api/`   | In discussion ([TOPIC-003](topics/TOPIC-003_openapi-documents-rendering-and-storage.md)) |
| Project Manager | `build-docs/project-mgr/`    | Initial spec drafted ([TOPIC-006](topics/TOPIC-006_project-mgr-product-management-and-work-tracking.md)): PRD, FSD, epics, milestones, release gates |
| WTS            | `build-docs/project-mgr/wts`  | Initial spec drafted (Work Tracking System, part of the Project Manager, modeled on the `backlog/` workspace) |

Other `build-docs/` subsystems that are specified together with the document domains:

- `cli/`: the distributed binary and the command surface.
- `mcp/`: the bundled MCP server, which gives document tools to LLMs.
- `static-ui/`: zero-build M3 web-component templates that render every document domain in a browser.

When you add a new domain spec under `build-docs/docs/`, use the depth and section structure of
`build-docs/docs/work-items/README.md`. The sections are: scope, hierarchy/taxonomy, lifecycle, document catalog,
directory layout, metadata standard, XPath query patterns, traceability, rules & constraints, revisions.

## Document standard being specified

All documents that `brevio` generates (after implementation) are strict XHTML (`xmlns="http://www.w3.org/1999/xhtml"`).
They must be well-formed for an XML parser (closed tags, quoted attributes, escaped entities). They must also render
natively in a browser through Material Web (`@material/web`) components, with no build step. The `<head>` of every
document must have seven universal baseline `<meta>` tags: `doc-id`, `doc-type`, `schema-version`, `domain`,
`status`, `created-at`, `updated-at`. It also has domain-specific metadata (for example `work-item-id`, `phase-id`,
`task-id`, `parent-doc-id`, `implements-req`, `depends-on`). This metadata makes a graph of links between documents that
a tool can query. In a consuming project, generated documents go in `.brevio/` at the repo root of that project (for
example `.brevio/work_items/{timestamp}_{slug}/...`). This layout follows the tier structure in
`build-docs/docs/work-items/README.md`.

When you write or review a new domain specification, apply this standard. New document types must use the same
metadata baseline and XHTML conformance rules. Do not introduce a different format.

## Programming guidelines

@build-docs/guidelines/README.md

When you write a design plan or an implementation plan, apply these guidelines. End each plan with a "Guideline check" section. Name the guideline numbers that shaped the plan, and name any guideline that the plan breaks, with the reason. If two guidelines conflict, the lower number wins.

These guidelines govern the code of `brevio`. They apply to plans and to the Go code in this repository.
