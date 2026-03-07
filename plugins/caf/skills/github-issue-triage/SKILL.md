---
name: github-issue-triage
description: >
  Run a GitHub issue triage session for the current repository. Use this skill whenever the user says
  "triage", "triage issues", "let's triage", "do a triage run", "triage GitHub issues", "triage my issues",
  or anything suggesting they want to go through open issues and classify/prioritize them. The skill finds
  all unlabeled issues, enriches each one with code context, rewrites the title and body in detail,
  classifies by type and priority, gets human approval, then writes the result back to GitHub. Issues
  that are too vague get a "needs-info" comment tagging the reporter. After triaging, offers to fix
  the issue immediately by routing bugs to superpowers:writing-plans, features to caf:planning, and
  enhancements to caf:planning.
  Also use when the user provides a list of feedback, notes, screenshots, or observations about the
  app — this skill will tease apart the items, classify them, create GitHub issues for each, and offer
  to route into the appropriate planning workflow. Use this skill even if the user only says
  "let's triage" or "can we go through the issues" — don't try to do this ad-hoc without the skill.
---

# GitHub Issue Triage

You are running a structured triage session. Work through unlabeled issues one at a time — enrich each
with real code context, present a proposed triage for human approval, then write it back to GitHub.
Nothing gets written to GitHub without explicit approval.

## Mode detection

At the start of every session, determine which mode applies:

**Repo triage mode** — activate when the user says "triage", "triage issues", "let's go through
the issues", or similar. Proceed to Phase 1 below.

**Feedback intake mode** — activate when the user provides a list of notes, observations,
bullets, or verbal feedback about the app (not a request to triage GitHub issues). Proceed to
the Feedback Intake section below.

**If ambiguous** — ask: "Do you want to triage existing GitHub issues in the repo, or process
a list of feedback into tracked items?"

---

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

After a successful write, prompt based on the type label:

- **Bug**: "Issue #[number] is triaged. Want to fix it now, or move to the next issue?"
- **Feature or Enhancement**: "Issue #[number] is triaged. Want to start planning this now, or move to the next issue?"
- **Documentation or Question**: "Issue #[number] is triaged. Moving to the next issue."
  *(No fix-now path for these types — continue automatically.)*

**Fix now — route based on type label:**

- **Bug** → Follow the `caf:planning` implementation handoff stage using issue `#[number]`
  as the task list location.
- **Feature (`feature-request`)** → Invoke `caf:planning` to start the feature requirements stage.
- **Enhancement** → Invoke `caf:planning` to start the enhancement stage.
- **Documentation** → Open the relevant documentation file and proceed directly.
- **Question** → No "fix now" path; move to the next issue.

**Later (Bug/Feature/Enhancement):** Continue to the next untriaged issue.

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

---

## Feedback Intake Mode

Use this mode when the user provides free-form feedback — bullet lists, verbal notes,
screenshots, or any mix of observations about the app.

### Step 1: Accept the input

Receive the feedback. It may arrive as:
- Pasted bullet points
- A verbal description in the conversation
- A mix of screenshots and notes
- A single GitHub issue that contains multiple conflated items

If the input is a GitHub issue body, treat it the same as free-form text — the goal is
to tease apart what's inside it.

### Step 2: Tease apart the items

Read through the input and identify each distinct item. A single bullet or sentence may
contain multiple items (e.g., "the sidebar is slow and the icons are wrong" = two items).

Present the items as a numbered list:

```
Here's what I found in your feedback — [N] distinct items:

1. [Item description — one sentence, specific]
2. [Item description]
...

Does this look right? Anything to merge, split, or add?
```

Wait for the human to confirm, adjust, or expand the list before proceeding.

### Step 3: Classify each item

For each confirmed item, assign:
- **Type**: bug / feature-request / enhancement / documentation / question
- **Priority**: P0 / P1 / P2 / P3 (use the same criteria as repo triage mode)
- **One-line rationale** for the type classification

Present all classifications at once:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PROPOSED CLASSIFICATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. [Item] → bug · P2: medium
   The sidebar renders slowly on large repos — existing functionality that's broken.

2. [Item] → enhancement · P3: low
   Adding keyboard shortcut to existing action — improvement to what's already there.

3. [Item] → feature-request · P2: medium
   New capability with no existing equivalent.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Do these classifications look right?
```

Wait for approval. Incorporate any corrections before proceeding.

### Step 4: Create GitHub issues

For each approved item, create a GitHub issue using the same enriched format as repo triage
mode (Steps 5-7 of Phase 3). Follow the same body template: Summary, Proposed Behavior or
Steps to Reproduce, Expected Behavior, Affected Files, Suggested Approach, Testing Requirements.

After creating each issue, confirm: "Issue #[number] created for [item]."

Note: GitHub issues are created for all types — bugs, features, enhancements, documentation, and questions alike.
The issue is the tracking artifact. The CAF planning spec is the planning artifact.
When a CAF spec is later created, it should reference the issue number, and the issue
should link to the spec.

### Step 5: Offer to start planning

After all issues are created, present a summary and offer to start planning:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
INTAKE COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Created:   [N] issues
  Bugs:          #[n], #[n]
  Features:      #[n]
  Enhancements:  #[n], #[n]
  Documentation: #[n]
  Questions:     #[n]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Want to start planning any of these now?
```

**If yes** — route based on type:
- Bug → Follow the `caf:planning` implementation handoff stage using the issue number as the task list location.
- Feature → `caf:planning` to start the feature requirements stage
- Enhancement → `caf:planning` to start the enhancement stage
- Documentation → open the relevant file and proceed directly
- Question → no planning path; the issue stands as a tracking item

**If no** — session complete.
