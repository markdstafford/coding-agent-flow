# Decomposing tasks

> **This is a process document.** Follow these steps in order.
> Refer to the planning skill's SKILL.md for shared concepts
> (prerequisites, checkpoints, wiki management).

This stage guides the breakdown of features into implementable, well-defined tasks.

## Output

Creates a `## Task list` section within the feature artifact containing:
- Hierarchical task breakdown (2-3 levels deep)
- Clear descriptions for each task
- Acceptance criteria
- Dependencies between tasks

## Process

### 1. Analyze the tech spec

Review the approved tech spec to identify:
- All components that need to be built
- APIs that need to be implemented
- Data models to create
- Tests required
- Documentation needed

### 2. Create task hierarchy

**Structure**: Use 2-3 levels:
- **Epic** — large body of work (optional, for very large features)
- **Story** — cohesive functionality (e.g., "Backend API for templates")
- **Task** — implementable unit of work (leaf level)

**Format**: GitHub-flavored Markdown with checkboxes:

```markdown
- [ ] **Epic: Email template management**
  - [ ] **Story: Backend API for templates**
    - [ ] **Task: Create `email_templates` table**
      - **Description**: Add database migration for email_templates table
      - **Acceptance criteria**:
        - [ ] Migration file created
        - [ ] Migration runs successfully on local database
        - [ ] Table schema matches tech spec
      - **Dependencies**: None
    - [ ] **Task: Implement POST /api/v1/templates endpoint**
      - **Description**: Create API route and controller for template creation
      - **Acceptance criteria**:
        - [ ] Endpoint returns 201 on success
        - [ ] Endpoint returns 4xx for invalid requests
        - [ ] Template persisted correctly in database
        - [ ] Unit tests pass with 100% coverage
      - **Dependencies**: "Task: Create `email_templates` table"
```

**Format is always required.** Smaller work produces fewer tasks, not a degraded format. A flat checklist (items without descriptions, acceptance criteria, and dependencies) is never acceptable output from this stage, regardless of feature size.

### 3. Write task details

Each leaf task MUST include:

**Description:**
- Clear, concise explanation of the work
- Enough detail for a junior developer to implement
- No ambiguity about what needs to be done

**Acceptance criteria:**
- Specific, testable conditions
- Use checkboxes for each criterion
- Cover functionality, testing, error handling
- Junior developer should know when the task is "done"

**Dependencies:**
- List other tasks that must complete first
- Use exact task names for clarity
- "None" if no dependencies

### 4. Validate task sizing

**Junior developer test:** Could a junior developer complete this task without asking questions?

If no → task is too large or ambiguous. Break down further or add more detail.

**Typical leaf task size:**
- 2-8 hours of work
- Single logical unit
- Clear start and end point

**CHECKPOINT**: Present a condensed summary for human approval — do not paste the full
artifact into chat. Use a two-level outline: stories as top-level bullets, leaf tasks as
sub-bullets under each story. Omit descriptions, acceptance criteria, dependencies, and
implementation detail from the chat presentation — these stay in the written file.

Example format:

```
- Story: Backend API for templates
  - Task: Create email_templates table
  - Task: Implement POST /api/v1/templates endpoint
- Story: Frontend template editor
  - Task: Build TemplateEditor component
  - Task: Wire save action to POST endpoint
```

If the human wants the full detail, offer: "Full plan is in `[artifact path]` — want me
to show it here?" Do not show the full text unless asked.

Get human approval on the condensed summary before proceeding to implementation handoff.

## Key guidelines

- You lead this (70% you, 30% human). Draft comprehensive task list, human reviews and refines.
- Start from tech spec components
- Work backwards from acceptance criteria
- Identify all prerequisite work
- Respect dependencies (database before API, etc.)
- List tasks in logical implementation order
- Group related tasks under stories

**Common task categories:**
- Data model / migrations
- API endpoints
- Business logic / services
- Frontend components
- Tests (unit, integration, E2E)
- Documentation
- Configuration / deployment

**Red flags — if you see these, fix them:**
- Task description is vague ("Make it work")
- No acceptance criteria
- Task seems to require multiple days
- Acceptance criteria are subjective
- Dependencies are unclear

## Implementation order

When implementing:
1. Tasks with no dependencies first
2. Mark tasks complete as acceptance criteria are met
3. Move to next available task (dependencies satisfied)
4. Update checkboxes in real-time
5. Add missed tasks as discovered during implementation

For examples, see:
- `references/examples/task-list.md` — complete task breakdown example
