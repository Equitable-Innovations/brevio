# CLI Specification

The `brevio` command line interface (CLI) is the distributed binary. It is the only component that reads and writes `brevio` documents. The MCP server calls the same code, so every operation behaves the same on both surfaces.

---

## Table of Contents

1. [Scope](#scope)
2. [Design Principles](#design-principles)
3. [Command Tree](#command-tree)
4. [Global Flags](#global-flags)
5. [Output and Exit Codes](#output-and-exit-codes)
6. [Document Commands](#document-commands)
7. [Work Item Commands](#work-item-commands)
8. [WTS Commands](#wts-commands)
9. [Project Manager Commands](#project-manager-commands)
10. [OpenAPI Commands](#openapi-commands)
11. [Server Commands](#server-commands)
12. [Setup Commands](#setup-commands)
13. [Concurrency and Atomic Writes](#concurrency-and-atomic-writes)
14. [Configuration](#configuration)
15. [Rules & Constraints](#rules--constraints)
16. [Revisions](#revisions)

---

## Scope

The CLI covers these items:

- The command tree and the flags that all commands share.
- The output format and the exit codes.
- Commands that read, validate, and query documents.
- Commands that change work items, the backlog, and project management documents.
- Commands that add OpenAPI sources and generate OpenAPI documents.
- The command that starts the MCP server.
- Commands that install hooks and agent assets.

The CLI does not define document formats. The [document system](../docs/README.md), the [Work Items domain](../docs/work-items/README.md), and the [project manager](../project-mgr/README.md) define them.

---

## Design Principles

1. The CLI works on local files under `.brevio/`. It makes no network call.
2. A command prints only what the caller asked for. An agent reads a small fragment, not a full document.
3. Every command that changes state does so atomically. A failed command leaves no partial change.
4. Every command can print JSON. A script or an agent never has to parse prose.
5. If a gate fails, the command that applies the gate exits with a non-zero code.

---

## Command Tree

```text
brevio
├── init                       # Create .brevio/ in the current project.
├── doc
│   ├── query                  # Run an XPath or JSON query on one document.
│   ├── validate               # Make sure that a document follows its schema.
│   └── schema                 # Print the schema of a document type.
├── work-item
│   ├── unit start | verify | complete
│   ├── ledger slice
│   └── hooks install
├── wts
│   ├── next-id | add | edit | remove | generate | list | check | start
├── pm
│   ├── init
│   ├── epic add | initiative add | milestone add
│   ├── edge add | edge remove
│   ├── graph | trace | gate | check
├── open-api
│   ├── add | update | generate | check | list | operation | schema | remove
├── wiki
│   ├── init | list | outline | get | find | rules | review | index | check
│   ├── page add | set-meta | move | remove
│   ├── section set | add | remove
│   ├── adr add | accept | deprecate
│   └── risk add | set
├── mcp                        # Start the MCP server on standard input and output.
└── assets install             # Install agent skills and agents.
```

---

## Global Flags

| Flag | Effect |
| :--- | :--- |
| `--root <path>` | Project root that holds `.brevio/`. The default is the nearest parent directory that holds `.brevio/`. |
| `--format <json\|text>` | Output format. The default is `text` on a terminal and `json` in a pipe. |
| `--quiet` | Prints nothing on success. |
| `--help` | Prints help for the command. |

---

## Output and Exit Codes

With `--format json`, a command prints one JSON object or one JSON object per line. An error prints one JSON object on standard error.

```json
{"error": "gate_failed", "message": "Unit unit_003_password_hasher has no passing verification.", "gate": "verification"}
```

| Exit code | Meaning |
| :--- | :--- |
| `0` | The command succeeded. |
| `1` | The command failed for an unexpected reason. |
| `2` | The arguments are not valid. |
| `3` | A document is not valid. It does not follow its schema. |
| `4` | A gate failed. The state did not change. |
| `5` | A drift scan found a difference. |
| `6` | A lock or version conflict stopped the write. |

---

## Document Commands

| Command | Effect |
| :--- | :--- |
| `brevio doc query --file <path> --xpath <expression>` | Prints the nodes of an XHTML document that match the expression. |
| `brevio doc query --file <path> --jq <expression>` | Prints the result of a JSON query on a JSON or JSONL file. |
| `brevio doc validate --file <path>` | Makes sure that the document is well-formed XML and carries the seven baseline tags. |
| `brevio doc schema --type <doc-type>` | Prints the schema of a document type. |

---

## Work Item Commands

These commands change the intent ledger of a work item. A worker agent never edits `03_intent_ledger.json` directly.

| Command | Effect |
| :--- | :--- |
| `brevio work-item unit start --unit-id <id>` | Moves a unit to `converging`. If a dependency is not `verified` or a hold is present, the command fails. |
| `brevio work-item unit verify --unit-id <id>` | Runs the verification contract of the unit and records the literal output. |
| `brevio work-item unit complete --unit-id <id> --verdict <pass\|fail> --exit-code <n> --tokens <n> --duration-ms <n>` | Records the result and the metrics. A `fail` verdict returns the unit to remediation and raises `retries`. |
| `brevio work-item ledger slice --unit-id <id>` | Prints the context slice of one unit. |
| `brevio work-item hooks install` | Writes the `build-gate` hook templates into the project. |

If `retries` exceeds the retry budget, the ledger places a `retry_budget_exceeded` hold. The next `brevio wts` status change reads the hold.

---

## WTS Commands

The [WTS specification](../project-mgr/wts/README.md#command-surface) defines the `brevio wts` commands.

| Command | Effect |
| :--- | :--- |
| `brevio wts next-id` | Prints the next free work item identifier. |
| `brevio wts add` | Creates a work item file and a backlog record. |
| `brevio wts edit` | Changes fields of a record and applies the gates. |
| `brevio wts remove` | Deletes a record and, by default, its file. |
| `brevio wts generate` | Rebuilds `backlog.md`. |
| `brevio wts list` | Prints records as JSON lines. |
| `brevio wts check` | Reports drift. It changes nothing. |
| `brevio wts start` | Seeds the runtime intent package. |

---

## Project Manager Commands

The [project manager specification](../project-mgr/README.md#command-surface) defines the `brevio pm` commands.

| Command | Effect |
| :--- | :--- |
| `brevio pm init` | Creates `prd.html`, `fsd.html`, and the directories. |
| `brevio pm epic add`, `brevio pm initiative add`, `brevio pm milestone add` | Create the document and register it. |
| `brevio pm edge add`, `brevio pm edge remove` | Change a `blocks` edge. Before `edge add` writes the edge, it makes sure that the edge creates no cycle. |
| `brevio pm graph` | Prints the work items that can start now. The flag `--reduce` reports redundant edges. |
| `brevio pm trace` | Rebuilds `traceability.html`. |
| `brevio pm gate --milestone <version>` | Runs the release gates and records the verdicts. The Authorization gate needs a human. |
| `brevio pm check` | Reports drift. It changes nothing. |

---

## OpenAPI Commands

The [OpenAPI specification](../docs/open-api/README.md#command-surface) defines the `brevio open-api` commands.

| Command | Effect |
| :--- | :--- |
| `brevio open-api add` | Saves a new OpenAPI source and generates its documents. |
| `brevio open-api update` | Replaces the source of an existing version and generates its documents. |
| `brevio open-api generate` | Regenerates the documents of one version or of every `stale` version. |
| `brevio open-api check` | Reports `stale` and `invalid` versions. It changes nothing. |
| `brevio open-api list` | Prints the APIs and versions from `catalog.json`. |
| `brevio open-api operation` | Prints sections of one operation document. |
| `brevio open-api schema` | Prints sections of one schema document. |
| `brevio open-api remove` | Deletes one API version and its catalog entry. |

---

## Wiki Commands

The [wiki specification](../docs/wiki/README.md#command-surface) defines the `brevio wiki` commands.

| Command | Effect |
| :--- | :--- |
| `brevio wiki init` | Creates `.brevio/wiki/`, the five core pages, and the index. |
| `brevio wiki list`, `brevio wiki outline`, `brevio wiki get` | Print page records, the sections of one page, or selected sections. |
| `brevio wiki find` | Searches the section text of published pages and prints section references. |
| `brevio wiki rules` | Prints the coding standard rules that apply to a set of paths. |
| `brevio wiki page add`, `set-meta`, `move`, `remove` | Create, change, move, or delete a freeform page. |
| `brevio wiki section set`, `add`, `remove` | Change one section. A write needs the expected section hash. |
| `brevio wiki adr add`, `accept`, `deprecate` | Create an ADR or change its status. `accept` and `deprecate` need a named human. |
| `brevio wiki risk add`, `set` | Add a risk or change its attributes. |
| `brevio wiki review` | Records a review of a page. |
| `brevio wiki index` | Makes sure that every page is valid and rebuilds `index.json` and `index.html`. |
| `brevio wiki check` | Reports invalid pages, broken links, index drift, and pages that need a review. It changes nothing. |

---

## Server Commands

| Command | Effect |
| :--- | :--- |
| `brevio mcp` | Starts the [MCP server](../mcp/README.md) on standard input and output. |

---

## Setup Commands

| Command | Effect |
| :--- | :--- |
| `brevio init` | Creates `.brevio/` and `.brevio/brevio.config.json`. |
| `brevio assets install` | Installs the agent skills and agents that the WTS specification lists. |

`brevio init` makes no change to a directory that already holds `.brevio/`. It prints a message and exits with code `2`.

---

## Concurrency and Atomic Writes

- A command that changes a ledger takes the file lock `.brevio/work_items/{id}/.ledger.lock` before it reads the ledger.
- The ledger carries `ledgerVersion`. A write must raise the version by one. If the versions do not match, the command retries with exponential backoff. After the last retry, the command exits with code `6`.
- A command that changes more than one file writes each file to a temporary name and renames all files at the end.

---

## Configuration

The file `.brevio/brevio.config.json` holds the project configuration. This version defines no keys.

---

## Rules & Constraints

1. This document represents the current state of the design. History lives in `topics/`.
2. The CLI is the only writer of `brevio` documents. The MCP server calls CLI code. A person can edit a wiki page or an OpenAPI source file in a text editor. `brevio wiki check` and `brevio open-api check` find an invalid edit.
3. A command that fails leaves every file unchanged.
4. A gate failure exits with code `4` and changes no state.
5. A command never lets an agent approve a `human-gate` or authorize a release.
6. A command never edits `backlog.md`, `traceability.html`, or a generated OpenAPI document in place. Only the generate commands write them. Only `brevio wiki` commands write the wiki `index.json` and `index.html`.
7. The CLI makes no network call.

---

## Revisions

| Date | Version | Description | Source |
| :--- | :--- | :--- | :--- |
| 2026-11-04 | 0.4.0 | Added the `brevio wiki` command group and the rule for hand edits of wiki pages. | TOPIC-002 |
| 2026-11-04 | 0.3.0 | Added the `brevio open-api` command group. | TOPIC-003 |
| 2026-11-04 | 0.2.0 | Removed the `brevio serve` command and the `serve.host` and `serve.port` configuration keys. | Removal of the REST API |
| 2026-11-01 | 0.1.0 | Initial specification: command tree, global flags, exit codes, document, work item, WTS, and project manager commands, server commands, concurrency, and configuration. | TOPIC-006 |
