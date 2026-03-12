# Implementation handoff

> **This is a process document.** Follow these steps in order.
> Refer to the planning skill's SKILL.md for shared concepts.

This stage hands off approved, decomposed work to an implementation system. The handoff is
extensible — currently the only supported implementation system is superpowers.

## When to use

After task decomposition is approved and the human is ready to begin implementation.

## Process

### 1. Confirm the implementation system

Check how many implementation systems are configured:

- **One system available**: announce it and proceed without prompting.
  Example: "Handing off to superpowers. Continuing…"
- **Multiple systems available**: present them as a numbered list and wait for a
  numeric response before continuing. Example:
  > "Ready to implement. Which system should we use?
  > 1. superpowers
  > 2. [other system]"

Currently superpowers is the only configured system, so in practice this step always
auto-selects and announces superpowers until a second system is added.

### 2. State the task list location

The task list location is known from context — state it explicitly so it can be passed to
the implementation system:

- **From CAF planning**: task list is in `.eng-docs/specs/[feature-filename].md`
- **From a GitHub issue**: task list is in GitHub issue `#N`

### 3. Ensure a GitHub issue exists

Check whether the feature or enhancement has a linked GitHub issue. For spec files, check
the frontmatter `issue` field. If no issue exists, create one:

```bash
body_file=$(mktemp)
printf '%s' "[summary of What and Why from spec, plus link to spec file]" > "$body_file"
output=$(gh issue create \
  --title "[feature/enhancement name]" \
  --label "[enhancement or feature-request],[priority-label]" \
  --body-file "$body_file")
rm "$body_file"
echo "$output"
```

The issue body should contain:
- A 1–2 sentence summary of what the feature/enhancement does and why it matters
- A link to the spec file: `**Spec:** .eng-docs/specs/[filename].md`

After creating the issue, record its number in the spec's frontmatter `issue` field.

If an issue already exists, note its number and continue.

### 4. Pull latest main

Before creating the worktree, ensure the local base branch is current:

```bash
git pull origin main
```

This ensures the new branch forks from the current tip of remote, not a stale local snapshot.

### 5. Create an isolated workspace

Invoke `superpowers:using-git-worktrees` to create a branch and worktree for this
implementation. Use a branch name derived from the feature or issue (e.g.
`feat/[short-name]` or `fix/[short-name]`).

**Do not proceed to step 6 if still on main after this step.** If the worktree skill
leaves the session on main, stop and investigate before continuing.

### 6. Implement with superpowers

Invoke `superpowers:writing-plans` with the following context. These instructions take
precedence over writing-plans' own defaults where they conflict.

**Task input.** The task list at [location] is the authoritative set of work to implement.
Read the full task list and its hierarchy from [location] before generating the plan. Use
the CAF tasks as the coarse-grained units — do not generate new top-level tasks from
scratch. You may (and should) expand each CAF task into TDD micro-steps, but the CAF task
is the unit of human-visible tracking.

**Plan storage.** Save the plan to `.eng-docs/.superpowers-plans/YYYY-MM-DD-<feature-name>.md`.
Before saving, check that `.eng-docs/.superpowers-plans/` is in `.gitignore`; add it if not.

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

**Finishing.** Invoke `superpowers:finishing-a-development-branch` with the following
context: this session is running inside a worktree. Present only these options:

1. Push and create a PR
2. Keep the worktree as-is to return later
3. Discard the work

Do not offer merge locally. Do not delete the worktree when pushing — keep it intact
until the PR merges.
