# Enhancement: Issue deduplication

## Parent feature

`feature-github-issue-triage.md`

## What

Before creating a new GitHub issue, the skill searches existing open issues for a close match. If a match is found, the evidence from the current item is appended to the existing issue rather than creating a duplicate. If no match is found, the skill proceeds to create a new issue as normal.

## Why

Triage sessions often produce near-duplicate issues — the same bug filed twice, or a friction log item that describes a problem already tracked from a previous session. Without deduplication, the backlog fills with redundant issues, triagers spend time manually cross-referencing, and the same work gets planned more than once.

By detecting duplicates at creation time and consolidating evidence onto the existing issue, the skill keeps the backlog clean without requiring the human to remember what was already filed.

## User stories

- Devon can see a duplicate warning before an issue is created, with the matching issue linked
- Devon can approve the match and have evidence appended to the existing issue instead of creating a new one
- Devon can reject the match and create a new issue anyway if the match is wrong
- Tara can run the same friction log through triage twice without generating duplicate issues
- Petra can see consolidated evidence on an issue when the same problem has been observed multiple times

## Design changes

*(Not applicable — no UI)*

## Technical changes

### Affected files

*(Populated during tech specs stage)*

### Changes

*(Added by tech specs stage)*

## Task list

*(Added by task decomposition stage)*
