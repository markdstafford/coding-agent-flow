# Scaling guide

The planning process is designed to scale from large applications to small bug fixes. Use this guide to determine which sections are required for different types of work.

## Scaling table

| Section | App | Large feature | Small feature | Enhancement | Big refactor | Small refactor | Chore/bug |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Name | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Description | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| What/why | Yes | Yes | No | Yes (brief, 1 para each) | Yes | No | No |
| Goals | Yes | Yes | Yes | No | No | No | No |
| High-level requirements | Yes | Yes | No | No | Yes | No | No |
| Personas | Yes | Yes | Yes | No (inherited) | No | No | No |
| Narratives | No | Yes | Yes | No (optional if novel) | No | No | No |
| User stories | No | Yes | Yes | Yes | No | No | No |
| Related features | Yes | Yes | Yes | No | Yes | Yes | Yes |
| Design spec | Optional | As needed | As needed | As needed (delta only) | As needed | No | No |
| Tech spec | Yes | Yes | Yes | Yes (delta only) | Yes | Yes | As needed |
| Task list | Yes | Yes | Yes | Yes | Yes | Yes | Yes |

## Task type definitions

### App

**Scope:** Entire application from scratch

**Required sections:**
- Name, Description
- What, Why, Goals
- High-level requirements
- Personas
- Related features (initial features list)
- Tech spec (high-level)
- Task list

**Notes:**
- This is the most comprehensive artifact
- Narratives not required at app level (they appear in features)
- High-level requirements are extensive (platform, data, security, operations, product)
- Sets constraints for all features

### Large feature

**Scope:** Significant new functionality with multiple user stories

**Examples:**
- Real-time collaboration system
- Payment processing integration
- Multi-tenant support

**Required sections:**
- Name, Description
- What, Why, Goals
- High-level requirements (feature-specific)
- Personas
- Narratives (multiple, 3-4 features revealed per narrative)
- User stories (5-10 per narrative)
- Related features
- Design spec (if UI work)
- Tech spec (comprehensive)
- Task list (potentially 20-50 leaf tasks)

**Notes:**
- Most comprehensive feature-level artifact
- All sections help ensure thorough planning
- High-level requirements focus on integration, data, security, operations for this feature

### Small feature

**Scope:** Focused functionality with clear, narrow scope

**Examples:**
- Export to CSV button
- Email notification for specific event
- Filter on existing list view

**Required sections:**
- Name, Description
- Goals (what success looks like)
- Personas (subset from app)
- Narratives (1-2 shorter narratives)
- User stories (3-8 stories)
- Related features
- Design spec (if UI work)
- Tech spec (may be brief)
- Task list (5-15 leaf tasks)

**Skip:**
- What/why (implied by goals and narratives)
- High-level requirements (uses app-level constraints)

**Notes:**
- Streamlined but still structured
- Goals ensure clear success criteria
- Narratives still reveal the feature naturally

### Enhancement

**Scope:** Improvement or extension to an existing feature

**Examples:**
- @-reference files in an existing chat panel
- Add keyboard shortcut to an existing action
- Add a filter to an existing list view

**Required sections:**
- Parent feature reference
- What, Why (1 paragraph each)
- User stories (3-6 stories)
- Technical changes (delta only)
- Task list

**Skip:**
- Goals (implied by Why)
- Personas (inherited from parent feature)
- Narratives (unless the interaction is genuinely novel)
- Non-goals (unless scope is ambiguous)
- Full architecture diagrams (reference parent's)

**Notes:**
- Everything is framed as delta on the parent feature
- Do not re-describe what already exists
- If the scope grows to feel like a new standalone feature, escalate to feature requirements

### Big refactor

**Scope:** Significant architectural or code structure changes

**Examples:**
- Migrate from REST to GraphQL
- Extract service from monolith
- Replace state management library

**Required sections:**
- Name, Description
- What, Why (explain the refactor rationale)
- High-level requirements (new architecture, dependencies, migration approach)
- Related features (affected features)
- Tech spec (comprehensive - architecture, migration plan, rollback strategy)
- Task list (broken into safe increments)

**Skip:**
- Goals (refactors typically don't have measurable KPIs)
- Personas, Narratives, User stories (no user-facing changes)
- Design spec (refactors are internal)

**Notes:**
- What/why critical to justify effort
- High-level requirements focus on technical constraints
- Tech spec must include migration and rollback strategies

### Small refactor

**Scope:** Focused code improvement without significant architectural change

**Examples:**
- Extract helper function
- Rename variable for clarity
- Consolidate duplicate code

**Required sections:**
- Name, Description
- Related features (what code/features affected)
- Tech spec (brief - what's changing and why)
- Task list (small, often 3-8 tasks)

**Skip:**
- What/why (described in description)
- Goals, High-level requirements
- Personas, Narratives, User stories
- Design spec

**Notes:**
- Minimal but ensures clarity
- Tech spec can be very brief (key algorithms section may suffice)
- Focus on clear task breakdown

### Chore/bug

**Scope:** Maintenance work or defect fix

**Examples:**
- Update dependency version
- Fix broken link
- Add missing error handling
- Update documentation

**Required sections:**
- Name, Description (describe the issue/work)
- Related features (what's affected)
- Task list (steps to complete)

**Optional:**
- Tech spec (only if complex - e.g., tricky algorithm fix)

**Skip:**
- What/why, Goals, High-level requirements
- Personas, Narratives, User stories
- Design spec

**Notes:**
- Minimal overhead for small tasks
- Description should clearly state the problem and solution
- For bugs: include steps to reproduce, expected vs actual behavior
- Task list ensures nothing is missed (even for "simple" bugs)

## How to decide

Use this decision tree:

1. **Is this a new application?** → App
2. **Is this user-facing functionality?**
   - **Improvement to something that already exists** → Enhancement
   - **Major functionality with multiple aspects** → Large feature
   - **Focused, single-purpose functionality** → Small feature
3. **Is this changing code structure/architecture?**
   - **Significant architectural change** → Big refactor
   - **Focused code cleanup** → Small refactor
4. **Is this maintenance or a bug fix?** → Chore/bug

## When in doubt

- **Start larger** - You can always streamline later
- **Can't decide between large and small feature?** → Start with large feature structure, skip sections that feel unnecessary
- **Human says "this is simple"?** → Ask clarifying questions about scope before choosing
- **Feeling like you're missing context?** → Go one level up in comprehensiveness

## Adapting during the process

It's okay to adjust the scale as you learn more:

- **Started as small feature but complexity emerged?** → Add back the skipped sections
- **Started as large feature but scope is narrower than thought?** → Remove unnecessary sections
- **Always document the decision** → Note in the artifact why certain sections were skipped

The goal is to have just enough structure to ensure quality without creating unnecessary overhead.
