# Enhancement template

Use this template when creating a new enhancement artifact.

```markdown
---
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: draft
issue: null
specced_by: github-username
implemented_by: null
superseded_by: null
---

# Enhancement: [short name]

## Parent feature

[Link to the feature this enhances — e.g. `feature-ai-chat-panel.md`]

## What

[1 paragraph: what this enhancement adds to the existing feature. Be specific about the new behavior, not the implementation.]

## Why

[1 paragraph: why this improvement matters, what gap it fills, what it unblocks for users.]

## User stories

- [Persona] can [new action enabled by this enhancement]
- [Persona] can [new action]

## Design changes

*(Added by design specs stage — frame as delta on the parent feature's design spec)*

## Technical changes

### Affected files

*(Populated during tech specs stage — list files that will change and why)*

- `path/to/file.ext` — [what changes and why]

### Changes

*(Added by tech specs stage — frame as delta on the parent feature's tech spec)*

## Task list

*(Added by task decomposition stage)*
```
