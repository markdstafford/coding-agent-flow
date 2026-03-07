---
name: github-issue-triage
description: >
  Run a GitHub issue triage session for the current repository. Use this skill whenever the user says
  "triage", "triage issues", "let's triage", "do a triage run", "triage GitHub issues", "triage my issues",
  or anything suggesting they want to go through open issues and classify/prioritize them. The skill finds
  all unlabeled issues, enriches each one with code context, rewrites the title and body in detail,
  classifies by type and priority, gets human approval, then writes the result back to GitHub. Issues
  that are too vague get a "needs-info" comment tagging the reporter. After triaging, offers to fix
  the issue immediately by writing a plan and creating a git worktree. Use this skill even if the user
  only says "let's triage" or "can we go through the issues" — don't try to do this ad-hoc without the skill.
  Also use when the user provides a list of feedback, notes, or observations about the app
  and wants to process them into discrete tracked items.
---

# GitHub Issue Triage

You are running a structured triage session. Work through unlabeled issues one at a time — enrich each
with real code context, present a proposed triage for human approval, then write it back to GitHub.
Nothing gets written to GitHub without explicit approval.

## Phase 1: Setup

### 1. Verify prerequisites
```bash
gh auth status
```
If `gh` is not authenticated or not installed, stop and tell the user.

### 2. Ask about analysis depth
Before fetching issues, ask once:

> "How thorough should the code analysis be for each issue?
> - **Quick** — keyword grep + read 2-3 relevant files (~1 min per issue)
> - **Thorough** — explore the full affected module, trace call chains, check test coverage (~3-5 min per issue)"

Remember the choice for the entire session.

### 3. Ensure required labels exist
Run `gh label list` and compare against the taxonomy in `references/labels.md`. Create any missing labels
using `gh label create "<name>" --color "<hex>" --description "<desc>"`. Refer to `references/labels.md`
for the exact colors and descriptions to use.

---

## Phase 2: Fetch Untriaged Issues

```bash
gh issue list --state open --json number,title,body,author,labels,createdAt \
  --jq '[.[] | select(.labels | length == 0)] | sort_by(.createdAt)'
```

- If there are **no untriaged issues**: tell the user and stop.
- If there are issues: tell the user how many were found (e.g. "Found 7 untriaged issues. Starting from oldest.") and begin.

---

## Phase 3: Process Each Issue

Work through issues sequentially. For each one:

### Step 1: Read the issue
Note the title, full body, and `author.login`.

### Step 2: Is this issue clear enough to triage?

An issue is **too unclear** if:
- The body is empty, a single line, or contains nothing actionable (e.g. "it broke", "doesn't work")
- There's not enough context to identify what part of the codebase is involved
- The expected vs. actual behavior is impossible to infer

If unclear → go to the **Unclear Issue Path** below.

### Step 3: Explore the codebase

**Quick mode:**
- Extract key terms from the issue title and body
- `grep -r` for those terms across the source tree
- Read the 2-3 most relevant files

**Thorough mode:**
- Map the relevant feature area by reading the directory structure
- Read source files, existing tests, and any related configuration
- Trace data flows or function call chains where helpful

Your goal: understand which specific files would need to change, and why.

### Step 4: Classify

Assign the following:

**One type label** (required):
| Label | When to use |
|-------|-------------|
| `bug` | Something is broken — behavior doesn't match intent |
| `feature-request` | A new capability being requested |
| `enhancement` | An improvement to existing functionality |
| `documentation` | Docs are missing, wrong, or unclear |
| `question` | User needs clarification, not a code change |

**One priority label** (required):
| Label | When to use |
|-------|-------------|
| `P0: critical` | Production broken, data loss, or security issue — fix immediately |
| `P1: high` | Significant user-facing breakage with no workaround |
| `P2: medium` | Bug with a workaround, or a high-value feature request |
| `P3: low` | Minor issue, polish, nice-to-have |

**Zero or more meta labels** (optional):
`usability`, `performance`, `security`, `good-first-issue`

### Step 5: Write the enriched title and body

**Title:** Always rewrite for precision. For bugs: describe what fails and when. For features: describe the capability being added.

**Body:** Use this template exactly:

```markdown
## Summary
[1-2 sentences. Clear, specific, no jargon unless necessary.]

## [Steps to Reproduce | Proposed Behavior]
[For bugs: numbered steps to trigger the issue.
For features: describe the desired behavior and why it's valuable.]

## Expected Behavior
[What should happen.]

## Actual Behavior
[For bugs only: what currently happens.]

## Affected Files
[Files likely requiring changes, based on code analysis.]
- `path/to/file.ext` — [reason this file is involved]

## Suggested Approach
[High-level implementation notes. What would need to change and roughly how.]

## Testing Requirements
[Specific tests that would verify the fix or feature is complete.]
- [ ] [test case description]
- [ ] [test case description]

---
*Original report by @[author.login]*
> [original title]
>
> [original body, quoted verbatim]
```

### Step 6: Present for approval

Show the proposed triage before touching anything:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROPOSED TRIAGE — Issue #[number]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TITLE WAS:  [original title]
TITLE NOW:  [new title]

LABELS:     [type label] · [priority label] · [meta labels if any]

[full new body]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Approve this triage? (yes / no / edit)
```

- **yes** → proceed to Step 7
- **no** → drop this issue and move to the next
- **edit** → incorporate the user's feedback and re-present; don't write until approved

### Step 7: Write back to GitHub

```bash
gh issue edit [number] \
  --title "[new title]" \
  --body "[new body]" \
  --add-label "[label1],[label2],..."
```

### Step 8: Offer to fix it now

After a successful write:

> "Issue #[number] is triaged. Want to fix it now, or move to the next issue?"

**Fix now — route based on type label:**

- **Bug** → Invoke `superpowers:writing-plans` to create a detailed implementation plan.
  Then invoke `superpowers:using-git-worktrees` to create an isolated worktree.
- **Feature (`feature-request`)** → Invoke `caf:planning` to start the feature requirements stage.
- **Enhancement** → Invoke `caf:planning` to start the enhancement stage.

**Later:** Continue to the next untriaged issue.

---

## Unclear Issue Path

When an issue is too vague to triage:

1. Post a comment tagging the reporter:
```bash
gh issue comment [number] --body "@[author.login] Thanks for filing this! To help us triage and prioritize it, could you share:

- A clear description of what's happening or what you'd like to see
- Steps to reproduce the issue (if it's a bug)
- What you expected vs. what actually happened

We'll hold off on triaging until we have a bit more context."
```

2. Add the `needs-info` label:
```bash
gh issue edit [number] --add-label "needs-info"
```

3. Tell the user: "Issue #[number] is unclear — tagged @[author.login] and added `needs-info`. Moving on."

4. Continue to the next issue.

---

## Phase 4: Session Summary

After all issues are processed:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TRIAGE SESSION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Triaged:      [n] issues
Needs info:   [n] issues (tagged for follow-up)
Fixing now:   [list issue numbers, or "none"]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Reference files

- `references/labels.md` — full label taxonomy with hex colors and descriptions for `gh label create`
