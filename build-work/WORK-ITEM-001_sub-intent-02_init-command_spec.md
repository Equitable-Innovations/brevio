# Sub-Intent 02 Specification: The `brevio init` Command

| Metadata | Value |
| :--- | :--- |
| `doc-id` | `WORK-ITEM-001:S02:SPEC` |
| `doc-type` | `sub-intent-spec` |
| `schema-version` | `1.0.0` |
| `intent-id` | `20261104T000000Z_go_project_skeleton_and_brevio_init` |
| `sub-intent-id` | `sub_intent_02_init_command` |
| `sub-intent-sequence` | 2 |
| `parent-doc-id` | `WORK-ITEM-001:02:DECOMP` ([decomposition](WORK-ITEM-001_intent-decomposition.md)) |
| `depends-on-sub-intents` | `sub_intent_01_module_and_root_command` |
| `total-units` | 2 |
| `assigned-agents` | `agent:go-engineer` (writer), `agent:code-reviewer` (approver) |
| `required-skills` | None |
| `status` | `pending` |
| `created-at` | 2026-11-04 |

---

## 1. Objective and Milestone Scope

Section id: `sub-intent-objective`.

- Primary objective: `brevio init` creates `.brevio/` and `.brevio/brevio.config.json`. If `.brevio/` exists, the command changes nothing, prints a message, and exits with code `2`.
- Architectural boundary: `internal/initcmd/`, `internal/cli/init.go`, and one change in `internal/cli/root.go` that adds the command.
- Prerequisite gate: `sub_intent_01_module_and_root_command` is `verified`. The root command, the printer, and the exit codes exist.
- Evidence this sub-intent delivers: `EV-001` and `EV-002`.

---

## 2. Context and "Do Not Touch" Constraints

Section id: `sub-intent-constraints`.

| Type | Entry |
| :--- | :--- |
| Write whitelist | `internal/initcmd/init.go`, `internal/initcmd/init_test.go`, `internal/cli/init.go`, `internal/cli/init_test.go`, `internal/cli/root.go` |
| `do-not-touch` | `build-docs/**`, `topics/**`, `backlog/**`, `build-work/**` |
| `do-not-touch` | `internal/cli/exitcode.go`, `internal/cli/errors.go`, `internal/cli/env.go`, `internal/cli/output.go` (the code from sub-intent 01) |
| Signature invariant | `NewRootCommand`, `Env`, `Printer`, and `ExitError` keep their exported signatures. |
| Inviolable | `internal/initcmd` imports only the standard library. It does not import `cobra` or `internal/cli`. |
| Inviolable | A failed run leaves every file unchanged. |
| Inviolable | No stub command for any group other than `init`. |

---

## 3. Referenced Patterns

Section id: `referenced-patterns`.

| ID | Source | What it defines |
| :--- | :--- | :--- |
| `PAT-001` | [CLI specification, Setup Commands](../build-docs/cli/README.md#setup-commands) | The effect of `brevio init` and the exit code `2` rule. |
| `PAT-002` | [CLI specification, Concurrency and Atomic Writes](../build-docs/cli/README.md#concurrency-and-atomic-writes) | A command that writes more than one file writes to temporary names and renames at the end. |
| `PAT-003` | [Solution strategy, `ADR-001` and `ADR-002`](WORK-ITEM-001_solution-strategy.md) | The temporary directory and rename pattern. The logic is separate from `cobra`. |
| `PAT-004` | Code of sub-intent 01: `internal/cli/root.go` | How a command receives `Env`, uses the `Printer`, and returns an `ExitError`. |

---

## 4. Unit Manifest

Section id: `unit-manifest`.

| Unit ID | Seq | Category | Single responsibility | Target files | `dependsOn` | Agent |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `unit_005_init_logic` | 1 | `backend` | `initcmd.Run` creates `.brevio/` and the configuration file in one atomic step. | `internal/initcmd/init.go`, `internal/initcmd/init_test.go` | None | `agent:go-engineer` |
| `unit_006_init_command_wiring` | 2 | `backend` | `brevio init` calls `initcmd.Run`, prints the result, and maps errors to exit codes. | `internal/cli/init.go`, `internal/cli/init_test.go`, `internal/cli/root.go` | `unit_005_init_logic` | `agent:go-engineer` |

Preconditions of the first unit: sub-intent 01 is `verified`. Postconditions of the last unit: the `EV-001` and `EV-002` commands exit with code `0`.

---

## 5. Edge Cases and Hazards

Section id: `hazards`.

| ID | Hazard | Guard |
| :--- | :--- | :--- |
| `HAZ-001` | A crash between two writes leaves `.brevio/` without its configuration file. | The temporary directory and rename pattern. A test forces the write to fail. |
| `HAZ-002` | A leftover temporary directory stays in the root after a failure. | `initcmd.Run` removes the temporary directory on every failure path. A test lists the root after a failure. |
| `HAZ-003` | The path `.brevio` exists as a file or as an empty directory. | Any existing `.brevio` entry counts as "already initialized". Tests cover both cases. |
| `HAZ-004` | The root is read-only. | The error is unexpected, so the exit code is `1`. No partial change remains. |
| `HAZ-005` | Exit code `2` also means "invalid arguments". | The JSON error code differs: `already_initialized` or `invalid_argument`. Tests make sure that both codes appear. |
| `HAZ-006` | `--root` is omitted. | For `brevio init`, the root is the working directory. The command does not search parent directories. |

---

## 6. Sub-Intent Verification Gate

Section id: `sub-intent-verification-gate`. Every command must exit with code `0`.

| Command | Expected output |
| :--- | :--- |
| `go build ./... && go vet ./... && go test ./...` | No output from build and vet. Every package line starts with `ok` or `?`. |
| `go build -o bin/brevio ./cmd/brevio && d=$(mktemp -d) && ./bin/brevio init --root "$d" --format text && test -f "$d/.brevio/brevio.config.json"` | One line: `Created .brevio/` (`EV-001`). |
| `d=$(mktemp -d) && ./bin/brevio init --root "$d" --format text && ./bin/brevio init --root "$d" --format text; test $? -eq 2` | The second run prints `.brevio/ already exists. No change made.` on standard error (`EV-002`). |
| `d=$(mktemp -d) && ./bin/brevio init --root "$d" --format json` | `{"created":".brevio"}` |
| `d=$(mktemp -d) && test -z "$(./bin/brevio init --root "$d" --quiet)" && test -d "$d/.brevio"` | No output. |
| `d=$(mktemp -d) && ./bin/brevio init --root "$d" --format text && test "$(cat "$d/.brevio/brevio.config.json")" = "{}"` | One line: `Created .brevio/`. |
| `! grep -r -e "net/http" -e "net\.Dial" --include=*.go .` | No output (`INV-002`). |
| `git diff --quiet 24a304e -- build-docs topics` | No output (`INV-003`). |

Forbidden output: a leftover directory whose name starts with `.brevio.tmp` in the root after any run.

Sign-off: `sub_intent_verification.html` must record `validation-verdict="pass"` before the ledger unlocks sub-intent 03.

---

## Revisions

| Date | Version | Description | Source |
| :--- | :--- | :--- | :--- |
| 2026-11-04 | 0.1.0 | Initial sub-intent specification. | WORK-ITEM-001 |
