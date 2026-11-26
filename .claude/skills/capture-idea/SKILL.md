---
name: capture-idea
description: "Instantly capture an idea (a feature, improvement, or \"we could...\") as a timestamped one-liner in topics/idea-stream.md, with no questions and no interruption. Use when the user says \"idea:\", \"capture an idea\", \"we could\", \"what if we\", \"park this idea\", or invokes /capture-idea. The LLM can also run it on its own when it has a useful idea while working on another task, so the idea is not lost."
metadata:
  node_type: memory
  originSessionId: 09e260c8-5e25-4d75-a62b-60bdd5d44d1e
  modified: 2026-11-26T00:02:06.121Z
---

# Capture Idea

Append one raw idea to `topics/idea-stream.md` in a single tool call. Then go back to the
current task. This skill must never pull the developer or the LLM away from the work in
progress.

An idea is a suggestion for something to build or change. It needs no discussion yet. If the
thing to capture is an open question or a design debate, use `capture-topic` instead.

## Steps

1. Take the idea text from the skill args or the user message. Keep their wording. Trim it
   to one line. Do not expand, interpret, or improve it. If no text exists, ask "What is
   the idea?" one time and wait.
2. Run the one Bash command below. If the file does not exist, the command makes it. Then
   the command reads the time and appends the entry. Replace `IDEA TEXT` with the idea. Do
   not run any other command first.

   ```bash
   f=topics/idea-stream.md
   [ -f "$f" ] || printf '# Idea Stream\n\nRaw ideas, captured fast with the `capture-idea` skill. Newest at the bottom.\nNo lifecycle. To discuss an idea, move it into a TOPIC-### file.\n\n---\n\n## Captured\n\n' > "$f"
   printf '%s\n' "- \`[$(date '+%Y-%m-%d %H:%M')]\` IDEA TEXT" >> "$f"
   ```

3. Reply with one short line, for example: `Idea captured.` Then continue the task that
   was in progress. If the LLM captured the idea itself, it adds at most one short sentence
   that names the idea.

## Rules

- Use one call to capture. Do not read the file, search the repo, or look for duplicates.
- Do not ask follow-up questions. Do not suggest next steps.
- Do not create a `TOPIC-###` file, edit `topic-registry.md`, or create a work item.
- If the user lists several ideas, write one bullet for each in the same command.
- If the user gave tags, put them at the end of the line as `#tag`. Do not add tags of your own.
- Never rewrite or reorder existing entries. Only append.
