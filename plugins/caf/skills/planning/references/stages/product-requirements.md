# Writing product requirements

> **This is a process document.** Follow these steps in order.
> Refer to the planning skill's SKILL.md for shared concepts
> (prerequisites, checkpoints, wiki management).

This stage guides the creation of product requirements for applications and features.

## Output artifacts

- `app.md` — application-level requirements (created once per project)
  - Contains: What, Why, Personas, Narratives, High-level requirements, Design guidance (optional), Related features
- `feature-*.md` — feature-level requirements (one per feature)
  - Contains: What, Why, Personas, Narratives, User stories, Goals/Non-goals
  - Design spec, Tech spec, and Task list sections are added by later stages

## For applications

### 1. What (1-3 paragraphs)

**Content**: What are we building, for whom, and what outcome?

**Process**:
- **If you have strong context**: Draft the What section yourself
- **If you're uncertain**: Ask the human for 1-2 sentences describing What, then expand
- Present your draft
- **CHECKPOINT**: Get human approval before proceeding to Why

**Anti-solutioning**: The What section describes the problem space and capability, NOT the implementation.
- What should answer: what can users do that they couldn't before?
- **Red flag**: If your What section mentions specific technologies, UI patterns, or architecture, you're solutioning. Remove those details.
- Before: "Episteme uses a Tauri-based desktop framework with React frontend to provide document authoring..."
- After: "Episteme is a document authoring application for teams working on collaborative documentation..."

### 2. Why (1-2 paragraphs)

**Content**: Why build this? How will users' lives change?

**Process**:
- **If you have strong context**: Draft the Why section yourself
- **If you're uncertain**: Ask the human for 1-2 sentences describing Why, then expand
- Present your draft
- **CHECKPOINT**: Get human approval on both What and Why before proceeding

**Anti-solutioning**: The Why section describes motivation and value, NOT how we'll deliver it.
- Why should answer: why does this matter to users/the business?
- **Red flag**: If your Why section reads like a pitch for a specific technical approach, you're solutioning. Focus on the user's problem.
- Before: "By using AI-powered document generation with vector embeddings, teams can..."
- After: "Writing good documentation requires deep thinking about critical decisions, but authors spend mental energy on mechanical tasks..."

### 3. Personas (3-5)

**Content**: Key user roles who will interact with the app

**Process**:
- Based on What/Why, propose 3-5 personas
- Format: "Name: Role" where name starts with same initial as role (e.g., "Dahlia: VP of deposit accounts")
- Present personas with brief description of each
- **CHECKPOINT**: Get human approval before proceeding

### 4. Narratives

**Content**: High-level user stories showing how personas use the app

**Process**:
- Ask human for 2-3 narrative titles or scenarios they want covered
- For each narrative:
  - Draft 2-3 paragraphs showing persona using the app (roughly ≤5 sentences per paragraph)
  - Focus on happy path, start-to-finish journey
  - Weave in interesting details that demonstrate specific capabilities
  - Each narrative should highlight 3-4 distinct features or interactions
  - Get approval before moving to next narrative
- **CHECKPOINT**: Get human approval on all narratives before proceeding

You lead drafting (70% you, 30% human), but human chooses the scenarios.

### 5. High-level requirements

**Content**: Platform, architecture, data, security, operations decisions

**Format**: Use terse, scannable bullets. Avoid prose — save details for ADRs and tech specs.
- **Good**: "Frontend: React 18, TypeScript, Vite"
- **Bad**: "The application will be built using a modern frontend stack with React 18 for the component model and TypeScript for type safety"

**Process**: Go through each subsection one at a time:

**a) Platform decisions**
- Ask about: Web/mobile/desktop, browser requirements, device support
- Draft platform requirements in terse bullet format
- Identify decisions that warrant ADRs
- **Checkpoint**: Get approval

**b) Architecture decisions**
- Ask about: Monolith vs microservices, frontend/backend separation, third-party services
- Identify ADR candidates: framework choices, build tools, state management, API style, styling approach, deployment strategy
- Draft architecture requirements in terse bullet format
- **Checkpoint**: Get approval; note ADRs for creation after app.md

**c) Data decisions**
- Ask about: Database type, data storage, data flow, multi-tenancy
- Identify ADR candidates: database choice, ORM/query builder, schema approach, caching, file storage
- Draft data requirements in terse bullet format
- **Checkpoint**: Get approval

**d) Security decisions**
- Ask about: Authentication, authorization, data protection, compliance
- Identify ADR candidates: auth method, authorization model, validation library
- Draft security requirements in terse bullet format
- **Checkpoint**: Get approval

**e) Operations decisions**
- Ask about: Monitoring, logging, deployment, scaling, backup
- Identify ADR candidates: monitoring tools, testing frameworks
- Draft operations requirements in terse bullet format
- **CHECKPOINT**: Get approval on complete high-level requirements section

**f) MANDATORY: ADR extraction checkpoint**

After completing all high-level requirements subsections, you MUST:

1. **Review all decisions** — go through each subsection and list all significant technology, framework, and architectural decisions
2. **Present ADR list** — show the human a list of decisions that need ADRs with brief rationale
3. **Create ADRs NOW** — before proceeding to features, work with the human to create these ADRs using the ADRs stage process. Do not skip this step.

Adapt subsections based on problem domain. Add or remove as needed.

### 6. Design guidance (optional)

**Content**: Typography, colors, spacing, component styles, icons

Only include if design constraints exist. Ask about brand colors, typography, spacing system, component preferences.

**CHECKPOINT**: Get approval

### 7. Related features

**Content**: List of planned/implemented features

- Ask human what features they envision
- Create list with brief descriptions
- **CHECKPOINT**: Get approval on complete app.md

## For features

> **Is this an enhancement?** If the work improves something that already exists rather than
> creating new standalone functionality, use the enhancements stage instead
> (`references/stages/enhancements.md`). The feature requirements stage is for new functionality.

### 1. What (1-3 paragraphs)

Same process as applications. Same anti-solutioning rules apply.

**CHECKPOINT**: Get human approval before proceeding to Why

### 2. Why (1-2 paragraphs)

Same process as applications. Same anti-solutioning rules apply.

**CHECKPOINT**: Get human approval on both What and Why before proceeding

### 3. Personas (subset from app)

- Reference personas from app.md
- Identify which ones use this feature (typically 1-3)
- **CHECKPOINT**: Get human approval

### 4. Narratives

- Ask human for 2-3 narrative titles or scenarios
- Draft each narrative one at a time (2-3 paragraphs, ≤5 sentences per paragraph)
- Focus on happy path, highlighting 3-4 distinct features per narrative
- **CHECKPOINT**: Get human approval on all narratives

### 5. User stories

- Review each narrative
- Extract 5-10 "[persona] can..." statements per narrative
- Present as bulleted lists grouped by narrative
- **CHECKPOINT**: Get human approval

### 6. Goals and non-goals

- Ask human about success metrics and what's out of scope
- Draft 3-5 measurable goals with specific targets
- Draft 0-5 non-goals (things explicitly not included)
- **CHECKPOINT**: Get human approval

### 7. High-level requirements (large features and big refactors only)

Most feature-*.md files will NOT have this section. Only include for features that introduce new architectural patterns, touch multiple systems, or involve significant refactors. See the planning skill's scaling guide for when this applies.

## Key guidelines

**Section-by-section approach:**
- **CRITICAL**: Never write multiple sections at once
- Complete one section, get approval, move to next
- If human wants to skip ahead, remind them of the process

**Effort split by section:**
- What/Why: You lead when confident (60/40), collaborate when uncertain (40/60)
- Personas: Proactive suggestion (50/50)
- Narratives: You lead drafting (70/30), human chooses scenarios
- User stories: You lead (60/40)
- Goals: Human leads (60% human, 40% you)

**Anti-solutioning checklist:**
- [ ] Does What describe the problem space and capability without prescribing implementation?
- [ ] Does Why describe motivation and value without pitching a technical approach?
- [ ] Are technologies, UI patterns, and architecture absent from What/Why sections?

For templates and examples, see:
- `references/templates/app.md` — template for app.md artifacts
- `references/templates/feature.md` — template for feature-*.md artifacts
- `references/examples/app.md` — complete app.md example
- `references/examples/feature.md` — complete feature example
