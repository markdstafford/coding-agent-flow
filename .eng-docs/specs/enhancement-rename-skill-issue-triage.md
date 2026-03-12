# Enhancement: Rename skill to issue-triage

## Parent feature

`feature-github-issue-triage.md`

## What

The skill is renamed from `caf:github-issue-triage` to `caf:issue-triage`. All internal references, trigger descriptions, and documentation are updated to use the new name. The feature spec is updated to reflect the rename and the intent to support platforms beyond GitHub in the future.

## Why

The skill is not exclusively a GitHub tool — it handles free-form feedback, friction logs, and verbal notes regardless of where issues are ultimately tracked. The `github-` prefix implies a narrower scope than the skill actually has, which discourages use in contexts where GitHub is not the obvious destination. A platform-neutral name better reflects what the skill does and leaves room for future extensibility.

## User stories

- Devon can invoke `caf:issue-triage` without having to remember it was previously named for GitHub specifically
- Petra can describe the triage skill to a new team member without the name implying it only works with GitHub issues

## Design changes

*(Added by design specs stage — frame as delta on the parent feature's design spec)*

## Technical changes

### Affected files

*(Populated during tech specs stage — list files that will change and why)*

### Changes

*(Added by tech specs stage — frame as delta on the parent feature's tech spec)*

## Task list

*(Added by task decomposition stage)*
