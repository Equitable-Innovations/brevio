---
name: capture-topic
description: "Capture a candidate topic (something that needs discussion, a design question, or a decision to make) into topics/thought-stream.md as a timestamped one-liner, so it can later be promoted to a TOPIC-### file. Use when the user says \"capture a topic\", \"topic:\", \"we need to discuss\", \"open question\", \"note for later\", or invokes /capture-topic. For plain ideas (a feature or improvement to build), use capture-idea instead."
metadata:
  node_type: memory
  originSessionId: 09e260c8-5e25-4d75-a62b-60bdd5d44d1e
  modified: 2026-11-26T00:02:17.423Z
---

# Capture Topic

Append a raw topic seed, as it is at this moment, to `topics/thought-stream.md`. This is a
light place to park something that needs discussion or a decision. It is not a `TOPIC-###`
file and is not part of any lifecycle. Later, the user can promote the entry into a real
`TOPIC-###` file. Record the topic with the fewest possible steps, and do not interrupt the user.

## Topic or idea

- A topic is a question, a debate, or a decision to make. Example: "Should the wiki use
  one XHTML file per page or one per section?" Use this skill.
- An idea is a suggestion to build or change something. Example: "We could add a dark
  theme to the static UI." Use `capture-idea`. It writes to `topics/idea-stream.md`.
- If it is not clear which one it is, ask the user one short question: "Topic or idea?"

## When to use

- The user raises something that needs discussion and wants to save it without a stop to
  develop it.
- Triggers: `/capture-topic`, "capture a topic", "topic:", "we need to discuss",
  "open question", "note for later".

## Steps

1. Get the topic text.
   - If the user supplied it (as skill args or in their message), use it word for word,
     with light cleanup: trim whitespace and make it a single line. Keep their wording and
     voice. Do not expand, interpret, or "improve" it.
   - If the user supplied nothing, ask: "What is the topic?" and wait.

2. Get the timestamp. Run `date "+%Y-%m-%d %H:%M"` (Bash tool). Use the local time of the
   project. Do not guess.

3. Append one list item to `topics/thought-stream.md`, at the end of the file (after
   the last existing entry / the `<!-- capture-topic appends below this line -->`
   marker). Format:

   ```
   - `[YYYY-MM-DD HH:MM]` <topic text on one line>
   ```

   - If the text has more than one sentence, keep it as one bullet. Do not split it into several.
   - If the user gave tags, you can append them as ` #tag` tokens at the end of the line.
   - Keep the existing file content exactly as it is. Only add the new line(s).

4. Confirm in a short reply. Show the one-line entry that you added and the file that it went
   into. Use one or two sentences. Do not summarize the whole file. Do not suggest next
   actions unless the user asks.

## Rules

- In this skill, never create a `TOPIC-###` file, change `topic-registry.md`, or change the status
  of any topic. Promotion to a real topic is a separate, deliberate step that the user controls.
- In this skill, never write ideas to `topics/idea-stream.md`. That file belongs to
  `capture-idea`.
- Write one entry for each invocation. If the user explicitly lists several distinct topics,
  write one bullet for each.
- Keep entries short. The goal is fast capture, not a complete record.
- If `topics/thought-stream.md` does not exist, create it first with this header:

  ```
  # Thought Stream

  Moment-in-time capture of raw topic seeds, to be revisited later.
  Not structured topics. Added via the `capture-topic` skill.

  ---

  ## Captured

  <!-- capture-topic appends below this line -->
  ```
