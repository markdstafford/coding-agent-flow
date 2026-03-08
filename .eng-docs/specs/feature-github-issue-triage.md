# GitHub issue triage

## What

GitHub issue triage is a structured workflow that guides an AI agent through the full cycle of GitHub issue management for a software project. It handles two distinct entry points: processing existing unlabeled issues already in the repository, and converting raw feedback into tracked GitHub issues.

In **repo triage mode**, the skill fetches all unlabeled open issues, explores the codebase to build context, and rewrites each issue's title and body to be precise and actionable. It classifies the issue by type and priority, presents the proposed triage for human approval, and writes the enriched result back to GitHub. Issues too vague to act on are tagged `needs-info` with a comment requesting clarification from the reporter.

In **feedback intake mode**, the skill accepts free-form input — bullet lists, verbal observations, friction logs, or any mix — and teases apart distinct items. It classifies each one, presents classifications for approval, then creates individual GitHub issues using the same enriched format. Friction logs receive special handling: pre-structured items are detected automatically, already-triaged items are skipped, and item severity is used as a priority signal.

In both modes, after each issue is triaged or created, the skill offers to route immediately into the appropriate planning workflow — bugs to implementation handoff, features and enhancements to `caf:planning`.

> **Note:** This skill is a candidate for renaming to "issue triage" with platform extensibility (GitHub, Linear, etc.) as a planned future enhancement.

## Why

GitHub issue triage solves a coordination problem: raw issues and feedback are noisy, inconsistently described, and hard to act on. Without a structured triage step, issues accumulate without priority, bugs get conflated with feature requests, and planning workflows start from unclear inputs.

By enriching issues with code context and enforcing a consistent format, the skill makes every issue actionable by anyone who picks it up — including AI agents handed a task list. By routing directly into planning workflows after triage, it removes the gap between "we know about this" and "someone is working on it."

## Personas

- **Devon: developer** — uses repo triage mode to clear an unlabeled issue backlog; uses feedback intake mode to convert a batch of notes from a testing session into tracked issues
- **Petra: product manager** — reviews proposed triages before they're written to GitHub; uses the planning routing to kick off requirements work immediately after triage
- **Tara: tester** — runs friction log sessions during workflow testing; hands the resulting log to the triage skill for conversion into GitHub issues

## Narratives

### Clearing the backlog

Devon has 12 unlabeled issues that have piled up over a sprint. He invokes the triage skill and chooses thorough analysis mode. For a report about the search bar returning no results, the skill greps for the relevant component, reads the search handler and its tests, and rewrites the issue with a precise title, root cause hypothesis, and suggested fix. Devon approves the triage, and the skill writes it back with `bug` and `P1: high` labels. When asked if he wants to fix it now, he says yes — the skill routes into the implementation handoff stage with the issue as the task list. By the end of the session, 10 issues are triaged, 2 are tagged `needs-info`, and two bug fixes are already in progress.

### Converting a friction log

Tara has just finished a testing session, guided by `caf:friction-log`. She ends up with 8 structured items on disk. She passes the file path to the triage skill, which reports "8 items found, 0 already triaged," and works through each one. Her two 🔴 blockers are proposed as P1, her 🟡 friction items as P2. As each issue is created in GitHub, the `Status` field in the friction log is updated to `triaged → #N`. Petra reviews the summary at the end and kicks off planning on the two P1 issues immediately.

## User stories

**Clearing the backlog**

- Devon can triage all unlabeled issues in a single session without leaving the terminal
- Devon can choose analysis depth to balance thoroughness against time
- Devon can review and approve each proposed triage before it's written to GitHub
- Devon can correct a proposed triage before it's written
- Devon can route a triaged bug directly into implementation without a separate planning step
- Petra can see a consistent issue format across all triaged items

**Converting a friction log**

- Tara can pass a friction log file path to the skill and have all untracked items converted to GitHub issues
- Tara can skip re-triaging items already linked to a GitHub issue
- Devon can see item severity translated into a priority prior for each friction log item
- Tara can see the friction log file updated with issue numbers as each item is created

## Goals

- Every triaged issue has exactly one type label and one priority label
- No issue is written to GitHub without explicit human approval
- Issues created in feedback intake mode are indistinguishable in quality from those produced in repo triage mode
- Friction log items are linked back to their source file after triage

## Non-goals

- Managing or closing existing issues — triage only handles unlabeled issues
- Assigning issues to team members
- Platform extensibility beyond GitHub — planned future enhancement
- Automated triage without human approval

## Design spec

*(Added by design specs stage)*

## Tech spec

*(Added by tech specs stage)*

## Task list

*(Added by task decomposition stage)*
