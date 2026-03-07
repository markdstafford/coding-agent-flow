# Implementation handoff

> **This is a process document.** Follow these steps in order.
> Refer to the planning skill's SKILL.md for shared concepts.

This stage hands off approved, decomposed work to an implementation system. The handoff is
extensible — currently the only supported implementation system is superpowers.

## When to use

After task decomposition is approved and the human is ready to begin implementation.

## Process

### 1. Confirm the implementation system

If the human has not already indicated a preference, ask:

> "Ready to implement. Which system should we use? (Currently available: superpowers)"

If the human has already said "implement with superpowers" or similar, skip the prompt.

### 2. State the task list location

The task list location is known from context — state it explicitly so it can be passed to
the implementation system:

- **From CAF planning**: task list is in `.eng-docs/specs/[feature-filename].md`
- **From a GitHub issue**: task list is in GitHub issue `#N`

### 3. Implement with superpowers

Invoke `superpowers:writing-plans` with the following context. These instructions take
precedence over writing-plans' own defaults where they conflict.

**Task input.** The task list at [location] is the authoritative set of work to implement.
Read the full task list and its hierarchy from [location] before generating the plan. Use
the CAF tasks as the coarse-grained units — do not generate new top-level tasks from
scratch. You may (and should) expand each CAF task into TDD micro-steps, but the CAF task
is the unit of human-visible tracking.

**Plan storage.** Save the plan to `.superpowers-plans/YYYY-MM-DD-<feature-name>.md`.
Before saving, check that `.superpowers-plans/` is in `.gitignore`; add it if not.

**Task checkbox tracking.** Each CAF leaf task's section in the plan must end with an
explicit step to check it off in [location]:

- **Markdown file**: edit the file — change `- [ ]` to `- [x]` for that task's line
- **GitHub issue**: read the current issue body, change `- [ ]` to `- [x]` for that
  task's line, write back with:
```bash
body_file=$(mktemp)
printf '%s' "$updated_body" > "$body_file"
gh issue edit [N] --body-file "$body_file"
rm "$body_file"
```

After checking off a leaf task, check whether all sibling tasks under the same parent are
now checked. If so, check off the parent the same way. Continue propagating up the
hierarchy until reaching a parent with unchecked children, or the root.

**Code review.** After all tasks complete and before invoking
`superpowers:finishing-a-development-branch`, run `gpt-code-review`.
