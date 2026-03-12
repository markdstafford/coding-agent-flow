# Enhancement: Handoff auto-selects single system

## Parent feature

*(Parent spec not yet created — caf:planning)*

## What

When the implementation handoff stage has only one routing system configured, it skips the selection prompt and announces the chosen system (e.g. "Handing off to superpowers…"). When multiple systems are configured, it presents them as a numbered list so the user can respond with a number rather than free text.

## Why

The current prompt asks "Which system should we use?" even when only one system is available, so the answer is always predetermined. This creates unnecessary friction with no decision to make. Presenting multiple options as a numbered list removes ambiguity and makes selection feel structured rather than informal.

## User stories

- Devon can complete an implementation handoff without answering a selection prompt when only one system is configured
- Devon can see which system was chosen automatically before handoff begins
- Devon can select from multiple handoff systems by typing a number when more than one is available

## Design changes

*(Added by design specs stage — frame as delta on the parent feature's design spec)*

## Technical changes

### Affected files

- `skills/planning/references/stages/implementation-handoff.md` — step 1 system-selection logic

### Changes

**Step 1 — Confirm the implementation system**

Replace the current unconditional prompt with a conditional:

- If one system is configured: announce it and proceed without prompting. Example: "Handing off to superpowers. Continuing…"
- If multiple systems are configured: present them as a numbered list and wait for a numeric response before continuing.

Currently superpowers is the only configured system, so in practice this change means step 1 always auto-selects and announces superpowers until a second system is added.

No other files, wiki documents, or ADRs are affected.

## Task list

- [x] **Story: Update system selection in implementation handoff**
  - [x] **Task: Replace unconditional system prompt with conditional logic**
    - **Description**: In `skills/planning/references/stages/implementation-handoff.md`, replace step 1's unconditional "Which system should we use?" prompt with a conditional. If one system is configured, announce it and proceed (e.g. "Handing off to superpowers. Continuing…"). If multiple systems are configured, present them as a numbered list and wait for a numeric response. Remove the existing "if the human has already said 'implement with superpowers'" clause — the new conditional covers that case.
    - **Acceptance criteria**:
      - [ ] Step 1 does not prompt the user when superpowers is the only system
      - [ ] Step 1 announces the auto-selected system before proceeding
      - [ ] Step 1 presents a numbered list when multiple systems are available
      - [ ] Numeric input is accepted for multi-system selection
    - **Dependencies**: None
