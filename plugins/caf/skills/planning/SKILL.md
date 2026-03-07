---
name: planning
description: >
  Use when creating new applications or features, adding functionality
  that needs planning, or doing any structured software development work
  including: writing product requirements, feature specs, or application
  specs; creating architectural decision records (ADRs); writing design
  specs for UI/UX; writing technical specifications; decomposing specs
  into implementation tasks. Also use when the user mentions "plan",
  "spec", "requirements", "ADR", or "break down work".
---

# Planning

A structured, collaborative process for planning software development work — from requirements through task decomposition.

## Core principles

- **Collaborative thinking** — each step builds on the previous through AI-human dialogue
- **Scalability** — adapts from full applications to small features (see references/scaling-guide.md)
- **Clarity through artifacts** — specific artifacts at each step ensure alignment
- **Iterative refinement** — every step and artifact can be revisited and refined

## When to use

- **DO use** — for any feature (use feature requirements stage), enhancement (use enhancement stage), or refactoring being added to the codebase
- **DO use** — when the user asks to "plan", "spec out", "create", or "add" something
- **DO use** — for both large and small features (scale the process to fit)
- **DON'T use the full planning process** — for bug fixes. Use the abbreviated bug workflow in `references/stages/bug-triage.md` instead, which produces a GitHub issue with root cause analysis and a task list.
- **DON'T use feature requirements** — for improvements to an existing feature; use the enhancement stage instead

## Process overview

### Initial setup (once per project)

1. **Create an app** — define the application's what, why, personas, and high-level requirements
2. **Extract and create ADRs** — document architectural decisions identified during app creation
3. **Initialize wiki** — create foundational documents for domain model, database schema, API contracts, and design system

### Per-feature workflow

1. **Specify feature requirements** — define what, why, goals, narratives, and user stories
2. **Create design spec** — design user flows and UI components (default for all features with UI; skip only for backend-only work)
3. **Write tech spec** — define architecture, APIs, data models, and implementation plan
4. **Decompose tasks** — break work into implementable tasks with acceptance criteria
5. **Implementation handoff** — route approved tasks to an implementation system

After completing the workflow, return to step 1 for the next feature.

## Shared concepts

These apply across ALL stages. Stage documents reference back here.

### Prerequisite checking

Before starting design specs, tech specs, or any stage that builds on prior work:

1. **Check for required ADRs** — verify foundational decisions exist in `.eng-docs/adrs/`
2. **Check wiki documents** — verify domain model, database schema, API contracts, and design system exist in `.eng-docs/wiki/`
3. **Check dependent features** — verify required feature specs exist in `.eng-docs/specs/`

**If CRITICAL gaps exist** (missing ADRs, no domain model, no database decisions):
- **STOP.** Do not proceed.
- Explain what's missing and why it matters.
- Propose creating the missing artifacts NOW.
- Do NOT offer to "document assumptions and proceed" for critical gaps.

**If minor gaps exist**: note them, ask whether to fill them first or document assumptions.

**If no gaps exist**: proceed normally, referencing existing artifacts.

### Section-by-section checkpoints

**Every stage follows this pattern:**

1. Complete one section of the artifact
2. Present it to the human for review
3. **CHECKPOINT**: Get explicit approval before proceeding to the next section
4. Never write multiple sections at once
5. If the human wants to skip ahead, remind them of the process; if they insist, note what was skipped

### Artifact file management

Before presenting the first section of any artifact:
1. Determine the file path using the artifact structure below
2. Create the file by copying from the appropriate template in `references/templates/`
3. Fill in the header fields that are already known (feature name, date, status: Draft)

After each section is approved, write it to the file immediately — do not wait for the session to end. The on-disk file is the canonical artifact. Chat output is a preview for discussion.

If the session ends unexpectedly mid-way through, the file should reflect all approved sections to that point.

### Wiki artifact management

Wiki documents in `.eng-docs/wiki/` are the source of truth. When any stage produces new:

- Domain entities → update `domain-model.md`
- Database tables/schemas → update `database-schema.md`
- API endpoints → update `api-contracts.md`
- UI components or design tokens → update `design-system.md`

### Working with your human

Key principles (see references/collaboration-protocol.md for full details):

- Enforce the process — keep humans accountable to each step
- Get What, Why, and Goals from the human — these require human input
- Human has final say on tech stack and architecture decisions
- Stay focused on the current step — don't jump ahead without consent
- Be a partner, not a sycophant — identify weaknesses and alternatives
- Ask clarifying questions rather than making assumptions

## Stage routing

When you identify which stage the user needs, **read the corresponding stage document and follow it as your process for that stage**:

| Stage | Read and follow | When |
|---|---|---|
| Create an app or feature requirements | `references/stages/product-requirements.md` | User wants to create an app, add a feature, write requirements |
| Write enhancement requirements | `references/stages/enhancements.md` | User wants to add to or improve an existing feature |
| Create ADRs | `references/stages/adrs.md` | Foundational decisions need documenting (technology, domain, schema) |
| Create design spec | `references/stages/design-specs.md` | Feature has UI elements that need design work |
| Write tech spec | `references/stages/tech-specs.md` | Translating requirements into technical architecture |
| Decompose tasks | `references/stages/task-decomposition.md` | Tech spec complete, need to break work into tasks |
| Implementation handoff | `references/stages/implementation-handoff.md` | Task list approved, ready to hand off to implementation system |
| Triage a bug and create a tracking issue | `references/stages/bug-triage.md` | A bug has been reported or discovered in feedback |

**Feature vs. enhancement:** If the work adds to or improves something that already exists (changing behavior, adding a capability to a known feature), use the enhancement stage. If it's new standalone functionality, use feature requirements.

**These stage documents contain imperative process instructions. Read and follow them step-by-step — they are not optional reference material.**

**Default to creating design specs** for any feature with UI elements. Only skip for truly backend-only work.

## Artifact structure

All artifacts are stored in `.eng-docs/`:

```
.eng-docs/
  adrs/                      # Architecture Decision Records
    adr-001-*.md             # Individual ADR files (numbered sequentially)
  specs/                     # Planning artifacts
    app.md                   # Application-level artifact (one per project)
    feature-*.md             # Feature-level artifacts (one per feature)
    enhancement-*.md         # Enhancement-level artifacts (one per enhancement)
  wiki/                      # Technical documentation (source of truth)
    domain-model.md          # Core domain entities and relationships
    database-schema.md       # Database tables, columns, relationships
    api-contracts.md         # API endpoint definitions and contracts
    design-system.md         # UI components, design tokens, patterns
```

**Initial setup**: When starting a new project, create stub files for wiki documents. These are populated as decisions are made and features are designed.

`.superpowers-plans/` is created by the implementation handoff stage and must be gitignored. It contains ephemeral implementation plans generated by superpowers and is not committed to the repository.

## Roles

You play different roles during the process. Each role has specific goals and tone. See references/roles.md for complete definitions.

Common patterns by stage:

- **Product requirements** — Product manager + Creative writer → Devil's advocate → Review
- **Design specs** — UX/UI designer + Brainstorm partner → Review
- **Tech specs** — Software engineer + Technical product manager → Devil's advocate → Review
- **Task decomposition** — Engineering manager + Software engineer → Review

## Scaling

Adapt the process based on task size. See references/scaling-guide.md for the complete scaling table and decision tree.

Quick guide:

- **App** — all sections, most comprehensive
- **Large feature** — all sections including narratives and user stories
- **Small feature** — goals, personas, narratives, user stories, tech spec, tasks (skip what/why)
- **Refactor** — what/why (if large), tech spec, tasks
- **Bug/chore** — description, tasks

**Task decomposition format is always required.** Smaller work produces fewer tasks — not a flat checklist. Every task decomposition, regardless of feature size, must use the full hierarchical format: stories grouping leaf tasks, each with a description, acceptance criteria, and dependencies. A flat checklist without these fields is never an acceptable substitute.

## Writing style

When drafting content for any planning artifact, follow the `caf:writing-guidelines` skill for style guidance including anti-jargon principles, tone by artifact type, and common mistakes.
