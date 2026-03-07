# Bug Triage

> **This is a process document.** Follow these steps in order.
> Refer to the planning skill's SKILL.md for shared concepts
> (prerequisites, checkpoints, wiki management).

This stage handles bug reports through codebase investigation, GitHub issue creation,
and abbreviated task decomposition. Unlike the full planning process, bugs skip
requirements, design, and tech spec stages. The investigation goes directly to
a GitHub issue and abbreviated task decomposition.

## When to use

When a bug is reported in user feedback, discovered during code review, or mentioned
in conversation. Triggered from the stage routing table in SKILL.md.

## Process

### 1. Investigate the codebase

Read the affected code to understand the root cause:
- Locate the relevant files and functions
- Trace the execution path that leads to the incorrect behavior
- Identify the specific lines or logic that need to change

Document:
- **Root cause**: the specific code defect
- **Affected files**: files that need changes
- **Proposed fix**: how to correct the behavior

**CHECKPOINT**: Summarize the root cause to the human. Confirm before proceeding.

### 2. Create a GitHub issue

Create a GitHub issue with the following body structure:

```bash
body_file=$(mktemp)
cat > "$body_file" << 'BODY'
## Summary
[1-2 sentences describing the bug and its impact.]

## Steps to Reproduce
1. [Step]
2. [Step]

## Expected Behavior
[What should happen.]

## Actual Behavior
[What currently happens.]

## Root Cause
[Specific code defect identified during investigation.]

## Affected Files
- `path/to/file.ext` — [reason]

## Suggested Approach
[How to fix it, based on investigation findings.]

## Testing Requirements
- [ ] [Specific verifiable test case]
- [ ] [Specific verifiable test case]
BODY
gh issue create \
  --title "[precise bug title]" \
  --label "bug,[priority-label]" \
  --body-file "$body_file"
rm "$body_file"
```

The "Testing Requirements" checkboxes become the leaf tasks for abbreviated task decomposition.

**CHECKPOINT**: Get human approval on the issue content before creating.

### 3. Abbreviated task decomposition

For each Testing Requirement checkbox, create a leaf task using the format from
`references/stages/task-decomposition.md`:

- **Description**: enough detail for a junior developer to implement without asking questions
- **Acceptance criteria**: specific, testable conditions as checkboxes
- **Dependencies**: explicit named references (or "None")

Smaller work has fewer tasks — never a flat checklist. The format is always required.

**CHECKPOINT**: Get human approval on the task list before proceeding.

### 4. Implementation handoff

Follow `references/stages/implementation-handoff.md` using the newly created GitHub issue
as the task list location.
