---
name: friction-log
description: >
  Use when the user wants to create a friction log or capture live usability feedback
  during a testing or workflow session. Triggered by any mention of "friction" or
  phrases like "log some friction", "capture feedback", or "document my experience".
  Not for triaging existing feedback — use caf:issue-triage for that.
---

# Friction Log

Guide the user through an interactive friction logging session. You are an active scribe —
the user narrates their experience and you structure it into a durable artifact.

## When to use

When a user is exercising a workflow and wants to capture pain points, confusion, and
friction for later triage. Friction logs are created by one person and triaged separately,
often by someone else.

## Process

### 1. Set up the file

Check if the current directory is inside a git repo:

```bash
git rev-parse --is-inside-work-tree 2>/dev/null
```

**If in a git repo:** Determine the file path: `.eng-docs/.friction-logs/YYYY-MM-DD-HHMMSS.md` using the current timestamp. Create `.eng-docs/.friction-logs/` if it doesn't exist. Create the file immediately with the stub header below — before saying anything to the user. Then tell the user: *"Creating friction log at `.eng-docs/.friction-logs/YYYY-MM-DD-HHMMSS.md`."*

**If not in a git repo:** Ask: *"Where should I create the friction log? (Default: `./friction-log-YYYY-MM-DD-HHMMSS.md`)"* Once the user provides a path — or accepts the default — create the file immediately at that location with the stub header below — before any further response.

Stub header to use:

```markdown
# Friction Log — YYYY-MM-DD HH:MM

---
```

(The header displays human-readable time; the filename uses compact sortable form `HHMMSS`.)

### 2. Accept items

Tell the user: *"Ready. Describe whatever friction you're experiencing — I'll structure it as we go."*

For each item the user raises:
1. Structure it into the friction item format (see below)
2. Ask clarifying questions if needed (see Clarifying questions)
3. Write the structured item to the file immediately
4. Confirm briefly (e.g. *"Got it, logged as Item 1."*) and wait for the next item

Keep the session flowing. The goal is low-friction logging.

### 3. End the session

When the user says "done", "that's it", "end the log", "finish", "stop", "quit", or similar:

1. Suggest a short kebab-case session name based on the items logged
   (e.g. `episteme-triage-flow`, `onboarding-setup`, `docs-first-run`)
2. Propose the rename: *"Rename to `<timestamp>-<session-slug>.md`?"*
   (Use the timestamp from the filename established in Step 1; place the renamed file in the same directory.)
3. On approval:
   ```bash
   mv <original-path> <directory>/<timestamp>-<session-slug>.md
   ```
   where `<original-path>` is the full path from Step 1 and `<directory>` is its parent directory.
4. Confirm: *"Saved to `<directory>/<timestamp>-<session-slug>.md`."*

---

## Friction item format

```markdown
## Item N: [one-line description]

**Trying to:** [what the logger was attempting — the goal, not the problem]
**What happened:** [narrative of the experience]
**Severity:** 🔴 blocker | 🟡 friction | 🟢 minor
**Suggested fix:** [optional — omit if none offered]
**Status:** untracked
```

Valid values: `untracked` (not yet in GitHub) or `triaged → #N` (linked to issue N). Always set to `untracked` when creating.

**Example:**

```markdown
## Item 5: Planning skill doesn't write artifacts to disk as it goes

**Trying to:** Track the spec document in my IDE as we worked through it section by section
**What happened:** Claude walked through each planning section in chat but wrote nothing to disk. I had to explicitly ask "can you write the doc to disk as you go?" mid-session — by that point I'd had no IDE view of the growing document and couldn't share it or verify it outside the conversation.
**Severity:** 🟡 friction
**Suggested fix:** Create the artifact file before the first section and write each approved section to disk immediately after checkpoint approval — not at session end.
**Status:** untracked
```

**Severity definitions:**
- 🔴 **blocker** — prevents completing the task; user would stop or switch tools
- 🟡 **friction** — annoying or confusing, but completable with effort
- 🟢 **minor** — small polish issue, unlikely to derail anyone

---

## Clarifying questions

Ask when ambiguity would make an item less useful for downstream triage. If you can
reasonably infer the intent or severity, do so.

**Good reasons to ask:**
- "Trying to" is completely unclear — what was the goal?
- Severity is ambiguous between 🔴 and 🟡 — did this block them completely?

**Don't ask:**
- For a suggested fix — user will offer one if they have it
- To confirm things already clearly stated

---

## Writing discipline

- Create the file before saying anything to the user
- Write each item to the file immediately after structuring it — never buffer
- If the session ends unexpectedly, all structured items to that point are on disk
- Follow `caf:writing-guidelines` when writing friction log items — clear, specific, jargon-free
