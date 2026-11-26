<!-- GENERATED FILE. Do not edit by hand. Source: backlog.jsonl. Template: backlog.template.md. Script: scripts/backlog.mjs -->
# Product & Development Backlog

This document is the central ledger that tracks all work items for **`brevio` (Token Optimized Software Harness)**.

A work item is the parent of all intent-driven SDLC documents. Each discrete unit of engineering work has its own work item file in this directory. The file name follows the pattern [`WORK-ITEM-###_{file-name-slug}.md`](README.md). Each file declares one intent. The index of work items is in [`backlog.jsonl`](backlog.jsonl). The script generates this file from that index.

For the work item lifecycle, the format standards, and the execution workflows, read the [Backlog Documentation](README.md).

---

## Backlog Metrics

| Total Work Items | Backlog | Planned | Ready | In Progress | In Review | Done | Blocked |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
{{METRICS_ROW}}

---

## Active & Planned Work Items

> Work items are sorted by Active Date, ascending. The work item that is active for the longest time is first. Work items in `BACKLOG` with `—` are last, sorted by Created Date, ascending. If two rows are equal, the ID decides the order.

| ID | Title | Status | Priority | Domain | Autonomy Tier | Assignee | Target Release | Active | Last Updated |
|:---|:---|:---|:---|:---|:---|:---|:---|:---:|:---|
{{ACTIVE_ROWS}}

---

## Completed & Archived Work Items

| ID | Title | Domain | Autonomy Tier | Resolution | Completed Date | Target Release |
|:---|:---|:---|:---|:---|:---|:---|
{{ARCHIVE_ROWS}}

---

## How to Register a New Work Item

> [!TIP]
> Never edit this file by hand. Each time `backlog.jsonl` changes, the script `backlog/scripts/backlog.mjs` rebuilds this file. To create a work item, run `/create-work-item` (or give the task to `backlog-manager`). To change its status, run `/update-work-item-status`. To report drift and apply bulk edits, run `/update-backlog`.

To register a work item manually:
1. Allocate the next ID. Run `node backlog/scripts/backlog.mjs next-id`.
2. Create the file. Copy [`template.md`](template.md) to `WORK-ITEM-###_{file-name-slug}.md` in this directory.
3. Declare the intent. Fill in the declared intent, invariants, boundary envelope, evidence requirements, and autonomy tier.
4. Register the record. Run `node backlog/scripts/backlog.mjs add --title "..." --domain CLI`. The script adds the line to `backlog.jsonl` and rebuilds this file.
5. Keep the lifecycle state current. As the work item moves from `READY` &rarr; `IN_PROGRESS` &rarr; `IN_REVIEW` &rarr; `DONE`, run `node backlog/scripts/backlog.mjs edit --id N --status IN_PROGRESS`.
6. Archive on completion. When you set the status to `DONE` or `CANCELLED`, the row moves to the Completed & Archived Work Items table.
