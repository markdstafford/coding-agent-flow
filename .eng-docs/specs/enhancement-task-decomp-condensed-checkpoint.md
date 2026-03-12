# Enhancement: Task decomposition condensed checkpoint

## Parent feature

*(Parent spec not yet created — caf:planning)*

## What

At the task decomposition review checkpoint, the skill presents a condensed summary rather than the full plan text. Stories appear as top-level bullets; leaf tasks appear as sub-bullets under each story. Acceptance criteria, dependencies, and implementation detail are omitted from the chat presentation but remain in the written artifact on disk. The full text is shown only if the user explicitly requests it.

## Why

The current checkpoint dumps the entire plan inline — every story, every leaf task, acceptance criteria, dependencies, and code snippets all at once. The result is a wall of text where the signal (what work is planned and in what order) is buried in the noise. A condensed summary lets the user confirm the shape of the work quickly, with the full detail available on disk for anyone who needs it.

## User stories

- Devon can review the task breakdown at a glance without scrolling through full implementation detail
- Devon can see stories and their leaf tasks in a two-level outline at the checkpoint
- Devon can request the full plan text if he wants to verify detail before approving
- Petra can approve a task decomposition without reading acceptance criteria she already reviewed in the tech spec

## Design changes

*(Added by design specs stage — frame as delta on the parent feature's design spec)*

## Technical changes

### Affected files

*(Populated during tech specs stage — list files that will change and why)*

### Changes

*(Added by tech specs stage — frame as delta on the parent feature's tech spec)*

## Task list

*(Added by task decomposition stage)*
