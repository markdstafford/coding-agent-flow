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

*(Populated during tech specs stage — list files that will change and why)*

### Changes

*(Added by tech specs stage — frame as delta on the parent feature's tech spec)*

## Task list

*(Added by task decomposition stage)*
