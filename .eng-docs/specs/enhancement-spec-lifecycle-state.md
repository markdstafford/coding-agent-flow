# Enhancement: Spec lifecycle state

## Parent feature

*(Parent spec not yet created — caf:planning)*

## What

CAF skills add a YAML frontmatter block to every spec, ADR, wiki, and friction log file they create. The frontmatter captures lifecycle state and authorship. For feature and enhancement specs, status moves through `draft | approved | implementing | complete | superseded`. For ADRs: `proposed | accepted | deprecated | superseded`. For wiki docs: `stub | active`. For friction logs: `untriaged | triaged`. Skills update `status` and `last_updated` automatically at key lifecycle events (approval, handoff, completion).

Unimplemented specs are stored in a `backlog/` subfolder under `.eng-docs/specs/` until implementation begins, at which point they move to the root `specs/` folder. This makes pending vs. in-progress work visible at the directory level without opening files.

## Why

As the number of specs grows, there is no signal at the directory level — or in the file itself — indicating whether a spec has been implemented. Determining status requires reading the file or digging through git history. A frontmatter block and subfolder structure let anyone (human or agent) understand the state of planning artifacts at a glance, without opening files.

## User stories

- Devon can see which specs are pending implementation by looking at the `backlog/` subfolder without opening any files
- Devon can see a spec's full lifecycle history (created, approved, implemented by whom) in the file's frontmatter
- Petra can tell at a glance whether a spec she is reviewing is a draft or approved
- An AI agent reading a spec can determine its status from frontmatter without inferring from file content
- Skills update status fields automatically so Devon does not have to maintain them manually

## Design changes

*(Added by design specs stage — frame as delta on the parent feature's design spec)*

## Technical changes

### Affected files

*(Populated during tech specs stage — list files that will change and why)*

### Changes

*(Added by tech specs stage — frame as delta on the parent feature's tech spec)*

## Task list

*(Added by task decomposition stage)*
