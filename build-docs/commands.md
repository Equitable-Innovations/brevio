# Command Reference

This document lists every `brevio` command and every MCP tool that the build documents define. It is an index. The owning specification defines the full behavior of each command.

---

## Table of Contents

1. [Global Flags](#global-flags)
2. [Exit Codes](#exit-codes)
3. [CLI Commands](#cli-commands)
4. [MCP Tools](#mcp-tools)
5. [Commands by Document Type](#commands-by-document-type)
6. [Revisions](#revisions)

---

## Global Flags

| Flag | Effect |
| :--- | :--- |
| `--root <path>` | Project root that holds `.brevio/`. The default is the nearest parent directory that holds `.brevio/`. |
| `--format <json\|text>` | Output format. The default is `text` on a terminal and `json` in a pipe. |
| `--quiet` | Prints nothing on success. |
| `--help` | Prints help for the command. |

Source: [CLI specification](cli/README.md#global-flags).

---

## Exit Codes

| Exit code | Meaning |
| :--- | :--- |
| `0` | The command succeeded. |
| `1` | The command failed for an unexpected reason. |
| `2` | The arguments are not valid. |
| `3` | A document is not valid. It does not follow its schema. |
| `4` | A gate failed. The state did not change. |
| `5` | A drift scan found a difference. |
| `6` | A lock or version conflict stopped the write. |

Source: [CLI specification](cli/README.md#output-and-exit-codes).

---

## CLI Commands

### Setup and Server

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio init` | Creates `.brevio/` and `.brevio/brevio.config.json`. If `.brevio/` exists, the command exits with code `2`. | [CLI](cli/README.md#setup-commands) |
| `brevio assets install` | Installs the agent skills and agents that the WTS specification lists. | [CLI](cli/README.md#setup-commands) |
| `brevio mcp` | Starts the MCP server on standard input and output. | [MCP](mcp/README.md#transport-and-startup) |

### Documents

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio doc query --file <path> --xpath <expression>` | Prints the nodes of an XHTML document that match the expression. | [CLI](cli/README.md#document-commands) |
| `brevio doc query --file <path> --jq <expression>` | Prints the result of a JSON query on a JSON or JSONL file. | [CLI](cli/README.md#document-commands) |
| `brevio doc validate --file <path>` | Makes sure that the document is well-formed XML and carries the seven baseline tags. | [CLI](cli/README.md#document-commands) |
| `brevio doc schema --type <doc-type>` | Prints the schema of a document type. | [CLI](cli/README.md#document-commands) |

### Work Items

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio work-item unit start --unit-id <id>` | Moves a unit to `converging`. If a dependency is not `verified` or a hold is present, the command fails. | [Intent Ledger](docs/work-items/intent-ledger.md) |
| `brevio work-item unit verify --unit-id <id>` | Runs the verification contract of the unit and records the literal output. | [Intent Ledger](docs/work-items/intent-ledger.md) |
| `brevio work-item unit complete --unit-id <id> --verdict <pass\|fail> --exit-code <n> --tokens <n> --duration-ms <n>` | Records the result and the metrics. A `fail` verdict returns the unit to remediation and raises `retries`. | [Intent Ledger](docs/work-items/intent-ledger.md) |
| `brevio work-item ledger slice --unit-id <id>` | Prints the context slice of one unit. | [Intent Ledger](docs/work-items/intent-ledger.md) |
| `brevio work-item hooks install` | Writes the `build-gate` hook templates into the project. | [CLI](cli/README.md#work-item-commands) |

### Work Tracking System (WTS)

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio wts next-id` | Prints the next free `WORK-ITEM-###` identifier. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts add` | Creates the work item file from the template, appends a record, and rebuilds `backlog.md`. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts edit` | Changes fields of a record. A status change applies the gates and the date stamps. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts remove [--keep-file]` | Deletes the record and the work item file. The flag `--keep-file` keeps the file. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts generate` | Rebuilds `backlog.md` without changing the data. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts list [--status <s>] [--id <id>]` | Prints records as JSON lines. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts check` | Reports drift between `backlog.jsonl` and the work item files. It changes nothing. | [WTS](project-mgr/wts/README.md#command-surface) |
| `brevio wts start` | Seeds the runtime intent package, and sets the `Runtime Directory` field. | [WTS](project-mgr/wts/README.md#command-surface) |

### Project Manager

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio pm init` | Creates `prd.html`, `fsd.html`, and the directories. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm epic add` | Creates an epic document and registers it. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm initiative add` | Creates an initiative document. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm milestone add` | Creates a milestone document. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm edge add` | Adds a `blocks` edge after the cycle check. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm edge remove` | Removes an edge. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm graph [--reduce]` | Prints the work items that can start now. The flag `--reduce` reports redundant edges. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm trace` | Rebuilds `traceability.html`. | [Project Manager](project-mgr/README.md#command-surface) |
| `brevio pm gate --milestone <version>` | Runs the release gates for a milestone and records the verdicts. The Authorization gate needs a human. | [CLI](cli/README.md#project-manager-commands) |
| `brevio pm check` | Reports drift between epics, work items, edges, and the matrix. It changes nothing. | [Project Manager](project-mgr/README.md#command-surface) |

### OpenAPI

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio open-api add --role <service\|client> --api-id <id> --source <file> [--major <n>]` | Copies the source into a new version directory, makes sure that it is valid, and generates the documents. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api update --api-id <id> --major <n> --source <file>` | Replaces the source of an existing version, makes sure that it is valid, and generates the documents. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api generate [--api-id <id>] [--major <n>]` | Makes sure that the sources are valid and regenerates the documents. With no flags, it regenerates every `stale` version. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api check` | Reports every `stale` or `invalid` version. It changes nothing. It exits with code `5` on a difference. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api list [--role <service\|client>]` | Prints the APIs and versions from `catalog.json`. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api operation --api-id <id> --major <n> (--slug <slug> \| --method <m> --path <p>) [--section <s>]` | Prints the selected sections of one operation document. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api schema --api-id <id> --major <n> --name <name> [--section <s>]` | Prints the selected sections of one schema document. | [OpenAPI](docs/open-api/README.md#command-surface) |
| `brevio open-api remove --api-id <id> --major <n>` | Deletes the version directory and its catalog entry. | [OpenAPI](docs/open-api/README.md#command-surface) |

### Wiki

| Command | Effect | Specification |
| :--- | :--- | :--- |
| `brevio wiki init` | Creates `.brevio/wiki/`, the five core pages with their required sections, and the index. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki list [--type <doc-type>] [--tag <tag>] [--review-due] [--drafts]` | Prints page records from the index. It prints no page bodies. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki outline --page <doc-id>` | Prints the sections of one page: identifier, title, token estimate, and hash. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki get --page <doc-id> [--section <id>[,<id>...]]` | Prints the selected sections of one page. The default is all sections. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki find --text <words> [--type <doc-type>] [--tag <tag>] [--limit <n>]` | Searches the section text of all published pages. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki rules --paths <path>[,<path>...] [--level <required\|recommended>]` | Prints the coding standard rules whose `data-applies-to` matches any of the paths. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki page add --slug <slug> --title <title> --summary <text> [--dir <dir>] [--tags <tags>]` | Creates a freeform page with one empty section and the status `in-progress`. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki page set-meta --page <doc-id> [--title] [--summary] [--tags] [--owner] [--covers-paths] [--status] [--nav-order]` | Changes the head tags of a page. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki page move --page <doc-id> --dir <dir>` | Moves a freeform page to another directory under `pages/`. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki page remove --page <doc-id> [--force]` | Deletes a freeform page. If a document references the page and `--force` is absent, the command fails with exit code `4`. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki section set --page <doc-id> --section <id> --file <fragment> --expect-hash <hash>` | Replaces the content of one section. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki section add --page <doc-id> --section <id> --title <title> --file <fragment> [--after <id>] [--parent <id>]` | Adds a new section. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki section remove --page <doc-id> --section <id> --expect-hash <hash>` | Removes a section. A required section of a core page cannot be removed. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki adr add --title <title> --slug <slug> [--intent-id <id>] [--supersedes <doc-id>]` | Creates an ADR with the status `proposed` and the next free number. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki adr accept --adr <doc-id> --by <identity>` | Sets `accepted`, `decided-at`, and `deciders`. The command needs a named human. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki adr deprecate --adr <doc-id> --by <identity>` | Sets `deprecated`. The command needs a named human. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki risk add --title <title> --likelihood <level> --impact <level> --owner <owner> --file <fragment>` | Adds a risk with the next free number, the status `open`, and a calculated rating. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki risk set --risk <RISK-NNN> [--likelihood] [--impact] [--status] [--owner]` | Changes the attributes of a risk and calculates the rating again. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki review --page <doc-id> --by <identity>` | Sets `reviewed-at` and `reviewed-by`. The command needs a named human. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki index` | Makes sure that every page is valid, then rebuilds `index.json` and `index.html`. | [Wiki](docs/wiki/README.md#command-surface) |
| `brevio wiki check [--max-age-days <n>]` | Reports invalid pages, missing sections, broken links, index drift, and `review_due` pages. If it finds a problem, it exits with code `5`. | [Wiki](docs/wiki/README.md#command-surface) |

---

## MCP Tools

The MCP server calls the same code as the CLI. Each tool returns JSON. Source: [MCP specification](mcp/README.md).

| Tool | Input | Matching CLI command |
| :--- | :--- | :--- |
| `brevio_query_doc` | `file`, and `xpath` or `jq` | `brevio doc query` |
| `brevio_get_schema` | `doc_type` | `brevio doc schema` |
| `brevio_validate_doc` | `file` | `brevio doc validate` |
| `brevio_ledger_slice` | `unit_id` | `brevio work-item ledger slice` |
| `brevio_update_unit` | `unit_id`, `action` (`start`, `verify`, `complete`), and for `complete`: `verdict`, `exit_code`, `tokens`, `duration_ms` | `brevio work-item unit start`, `verify`, `complete` |
| `brevio_get_gate_state` | `unit_id` | None |
| `brevio_wts_list` | Optional `status`, `id` | `brevio wts list` |
| `brevio_wts_add` | Work item fields | `brevio wts add` |
| `brevio_wts_edit` | `id` and the fields to change | `brevio wts edit` |
| `brevio_wts_start` | `id` | `brevio wts start` |
| `brevio_wts_check` | None | `brevio wts check` |
| `brevio_pm_get_requirement` | `id` (`PRD-REQ-###` or `FSD-FR-###`) | None |
| `brevio_pm_graph` | Optional `reduce` | `brevio pm graph` |
| `brevio_pm_edge` | `action` (`add` or `remove`), `source`, `target` | `brevio pm edge add`, `remove` |
| `brevio_pm_trace` | Optional `requirement` | None (reads the matrix) |
| `brevio_pm_gate_state` | `milestone` | None (reads gate results) |
| `brevio_openapi_list` | Optional `role`, `api_id` | `brevio open-api list` |
| `brevio_openapi_find` | `api_id`, `major`, and one of `path_contains`, `tag`, `method` | None |
| `brevio_openapi_get_operation` | `api_id`, `major`, `slug` or `method` and `path`, optional `sections` | `brevio open-api operation` |
| `brevio_openapi_get_schema` | `api_id`, `major`, `name`, optional `sections` | `brevio open-api schema` |
| `brevio_openapi_check` | None | `brevio open-api check` |
| `brevio_wiki_list` | Optional `doc_type`, `tag`, `review_due`, `drafts` | `brevio wiki list` |
| `brevio_wiki_outline` | `page` | `brevio wiki outline` |
| `brevio_wiki_get` | `page`, optional `sections` | `brevio wiki get` |
| `brevio_wiki_find` | `text`, optional `doc_type`, `tag`, `limit` | `brevio wiki find` |
| `brevio_wiki_rules` | `paths`, optional `level` | `brevio wiki rules` |
| `brevio_wiki_set_section` | `page`, `section`, `content`, `expect_hash`, and for a new section `title` and optional `after` or `parent` | `brevio wiki section set`, `add` |
| `brevio_wiki_add_page` | `slug`, `title`, `summary`, optional `dir`, `tags` | `brevio wiki page add` |
| `brevio_wiki_add_adr` | `title`, `slug`, optional `intent_id`, `supersedes` | `brevio wiki adr add` |
| `brevio_wiki_add_risk` | `title`, `likelihood`, `impact`, `owner`, `content` | `brevio wiki risk add` |
| `brevio_wiki_check` | Optional `max_age_days` | `brevio wiki check` |

Each of these actions needs a named human, so the MCP server provides no tool for them:

- Accept or deprecate an ADR.
- Set a risk to `accepted` or `closed`.
- Mark a page as reviewed.
- Approve a `human-gate`.
- Authorize a release.

---

## Commands by Document Type

This table shows which command group creates, changes, or reads each document family.

| Document family | Commands that write it | Commands that read it |
| :--- | :--- | :--- |
| Work item intent package and intent ledger | `brevio wts start`, `brevio work-item unit ...` | `brevio work-item ledger slice`, `brevio doc query` |
| Backlog index and work item files | `brevio wts add`, `edit`, `remove` | `brevio wts list`, `check` |
| Backlog view (`backlog.md`) | `brevio wts generate` | None |
| PRD, FSD, epics, initiatives, milestones | `brevio pm init`, `epic add`, `initiative add`, `milestone add`, `gate` | `brevio pm graph`, `check`, `brevio doc query` |
| Traceability matrix | `brevio pm trace` | `brevio doc query` |
| OpenAPI sources and generated documents | `brevio open-api add`, `update`, `generate`, `remove` | `brevio open-api list`, `operation`, `schema`, `check` |
| Wiki pages, ADRs, and risks | `brevio wiki init`, `page ...`, `section ...`, `adr ...`, `risk ...`, `review` | `brevio wiki list`, `outline`, `get`, `find`, `rules`, `check` |
| Wiki index | `brevio wiki index` | `brevio wiki list` |

---

## Revisions

| Date | Version | Description | Source |
| :--- | :--- | :--- | :--- |
| 2026-11-04 | 0.1.0 | Initial consolidated list of CLI commands and MCP tools. | CLI, MCP, WTS, Project Manager, OpenAPI, and Wiki specifications |
| 2026-11-25 | 0.1.1 | Rewrote the prose in plain English. No change to the design. | User request |
