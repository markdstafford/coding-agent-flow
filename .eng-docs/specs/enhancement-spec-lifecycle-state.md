# Enhancement: Spec lifecycle state

## Parent feature

*(Parent spec not yet created — caf:planning)*

## What

CAF skills add a YAML frontmatter block to every spec, ADR, wiki, and friction log file they create. The frontmatter captures lifecycle state and authorship. For feature and enhancement specs, status moves through `draft | approved | implementing | complete | superseded`. For ADRs: `proposed | accepted | deprecated | superseded`. For wiki docs: `stub | active`. For friction logs: `untriaged | triaged`. Skills update `status` and `last_updated` automatically at key lifecycle events (approval, handoff, completion).

Unimplemented specs are stored in a `backlog/` subfolder under `.eng-docs/specs/` until implementation begins, at which point they move to the root `specs/` folder. This makes pending vs. in-progress work visible at the directory level without opening files.

## Why

As the number of specs grows, there is no signal at the directory level — or in the file itself — indicating whether a spec has been implemented. Determining status requires reading the file or digging through git history. A frontmatter block and subfolder structure let anyone (human or agent) understand the state of planning artifacts at a glance, without opening files.

## User stories

- Devon can see which specs are pending implementation by looking at the `backlog/` subfolder without opening any files
- Devon can see a spec's full lifecycle history (created, approved, implemented by whom) in the file's frontmatter
- Petra can tell at a glance whether a spec she is reviewing is a draft or approved
- An AI agent reading a spec can determine its status from frontmatter without inferring from file content
- Skills update status fields automatically so Devon does not have to maintain them manually

## Design changes

*(Added by design specs stage — frame as delta on the parent feature's design spec)*

## Technical changes

### Affected files

**Templates:**
- `references/templates/feature.md` — add YAML frontmatter block
- `references/templates/enhancement.md` — add YAML frontmatter block
- `references/templates/adr.md` — add YAML frontmatter block
- `references/templates/app.md` — add YAML frontmatter block (no status field)
- `references/templates/wiki/api-contracts.md` — add YAML frontmatter block
- `references/templates/wiki/database-schema.md` — add YAML frontmatter block
- `references/templates/wiki/design-system.md` — add YAML frontmatter block
- `references/templates/wiki/domain-model.md` — add YAML frontmatter block

**Stage documents:**
- `references/stages/product-requirements.md` — create feature spec in `backlog/` with frontmatter; set wiki docs to `active` on first substantive edit
- `references/stages/enhancements.md` — create enhancement spec in `backlog/` with frontmatter
- `references/stages/adrs.md` — create ADR with frontmatter; set `decided_by` and update `status` to `accepted` at approval checkpoint
- `references/stages/task-decomposition.md` — update spec `status` to `approved` at checkpoint
- `references/stages/implementation-handoff.md` — move spec from `backlog/` to `specs/`; set `status: implementing` and `implemented_by`; set `status: complete` after PR is created

**Skill documents:**
- `planning/SKILL.md` — update artifact structure section to document `backlog/` and frontmatter convention
- `skills/friction-log/SKILL.md` — canonicalize the frontmatter schema (schema exists in practice; make it normative)

### Changes

**Frontmatter schemas**

Feature and enhancement specs:

```yaml
---
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: draft
issue: null
specced_by: github-username
implemented_by: null
superseded_by: null
---
```

Valid status values: `draft | approved | implementing | complete | superseded`

ADRs:

```yaml
---
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: proposed
decided_by: null
superseded_by: null
---
```

Valid status values: `proposed | accepted | deprecated | superseded`

The existing `## Status` section in ADR files is kept — it remains useful in rendered Markdown. Frontmatter `status` is the machine-readable canonical field; the `## Status` section is the human-readable narrative.

`app.md`:

```yaml
---
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
---
```

Wiki documents:

```yaml
---
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: stub
---
```

Valid status values: `stub | active`

Friction logs:

```yaml
---
created: YYYY-MM-DD HH:MM
last_updated: YYYY-MM-DD
status: untriaged
captured_by: github-username
triaged_by: null
---
```

Valid status values: `untriaged | partially-triaged | triaged`

Person fields support YAML list syntax when multiple people are involved:

```yaml
specced_by:
  - alice
  - bob
```

`created` is sourced from the file's first git commit. `superseded_by` holds the filename of the superseding artifact (e.g. `enhancement-new-thing.md`). Skills update `last_updated` on every edit to any artifact file.

**`backlog/` subfolder**

New feature and enhancement specs are created in `.eng-docs/specs/backlog/` instead of `.eng-docs/specs/`. At implementation handoff the spec moves to `.eng-docs/specs/`. ADRs and wiki documents go directly to their canonical locations — no `backlog/` staging.

**Lifecycle transitions**

| Event | Stage doc | Status before | Status after | Other fields changed |
|---|---|---|---|---|
| Spec created | `enhancements.md`, `product-requirements.md` | — | `draft` (in `backlog/`) | `specced_by` set |
| ADR created | `adrs.md` | — | `proposed` | — |
| ADR accepted | `adrs.md` | `proposed` | `accepted` | `decided_by` set |
| Wiki doc created | `product-requirements.md` | — | `stub` | — |
| Wiki doc first substantive edit | any stage | `stub` | `active` | — |
| Task decomposition approved | `task-decomposition.md` | `draft` | `approved` | — |
| Implementation handoff begins | `implementation-handoff.md` | `approved` | `implementing` (moved to `specs/`) | `implemented_by` set |
| PR created | `implementation-handoff.md` | `implementing` | `complete` | — |

`last_updated` is updated on every edit and is not listed separately per row.

**`planning/SKILL.md` artifact structure update**

Add to the artifact structure section:
- `specs/backlog/` — unimplemented specs; moved to `specs/` at implementation handoff
- A note that all artifact files begin with a YAML frontmatter block; `status` reflects lifecycle state

## Task list

- [x] **Story: Add frontmatter to templates**
  - [x] **Task: Add frontmatter to spec templates**
    - **Description**: Add the feature/enhancement YAML frontmatter block to `references/templates/feature.md` and `references/templates/enhancement.md`. Place the block at the top of the file, above the `# [title]` heading inside the fenced markdown code block in each template.
    - **Acceptance criteria**:
      - [ ] `feature.md` template has frontmatter block with `created`, `last_updated`, `status: draft`, `issue: null`, `specced_by`, `implemented_by: null`, `superseded_by: null`
      - [ ] `enhancement.md` template has the same frontmatter block
      - [ ] Frontmatter appears above the `#` heading inside the template's fenced code block
    - **Dependencies**: None
  - [x] **Task: Add frontmatter to ADR template**
    - **Description**: Add the ADR YAML frontmatter block to `references/templates/adr.md`. Place it above the `# ADR NNN:` heading. The existing `## Status` section is kept as-is — do not remove it.
    - **Acceptance criteria**:
      - [ ] `adr.md` has frontmatter block with `created`, `last_updated`, `status: proposed`, `decided_by: null`, `superseded_by: null`
      - [ ] Frontmatter appears above the `# ADR NNN:` heading
      - [ ] Existing `## Status` section is unchanged
    - **Dependencies**: None
  - [x] **Task: Add frontmatter to app.md template**
    - **Description**: Add the app.md YAML frontmatter block to `references/templates/app.md`. Place it above the first heading. No `status` field — app.md has no lifecycle state.
    - **Acceptance criteria**:
      - [ ] `app.md` template has frontmatter block with `created` and `last_updated` only
      - [ ] No `status` field present
      - [ ] Frontmatter appears above the first heading inside the template
    - **Dependencies**: None
  - [x] **Task: Add frontmatter to wiki templates**
    - **Description**: Add the wiki YAML frontmatter block to all four wiki templates: `references/templates/wiki/api-contracts.md`, `database-schema.md`, `design-system.md`, `domain-model.md`. Place each block at the top of the file, above the first heading.
    - **Acceptance criteria**:
      - [ ] All four wiki templates have frontmatter with `created`, `last_updated`, `status: stub`
      - [ ] Frontmatter appears above the first heading in each file
    - **Dependencies**: None

- [x] **Story: Update stage docs — artifact creation**
  - [x] **Task: Update `enhancements.md` — create spec in `backlog/` with frontmatter**
    - **Description**: In `references/stages/enhancements.md`, update step 1 (the file creation instruction) to: (1) use path `.eng-docs/specs/backlog/enhancement-[name].md` instead of `.eng-docs/specs/enhancement-[name].md`, and (2) instruct the skill to populate the frontmatter block when creating the file, setting `created` from today's date, `status: draft`, and `specced_by` from the current GitHub username.
    - **Acceptance criteria**:
      - [ ] Step 1 specifies `backlog/` as the creation path
      - [ ] Step 1 instructs populating `created`, `status: draft`, and `specced_by` at file creation
      - [ ] `last_updated` is set to the same value as `created` on file creation
    - **Dependencies**: Task: Add frontmatter to spec templates
  - [x] **Task: Update `product-requirements.md` — create spec in `backlog/` with frontmatter; set wiki docs to `active` on first substantive edit**
    - **Description**: In `references/stages/product-requirements.md`, update the feature spec creation step to use `.eng-docs/specs/backlog/feature-[name].md` and populate frontmatter on creation. Also add an instruction that when any wiki document (`domain-model.md`, `database-schema.md`, etc.) receives its first substantive content, the skill updates that file's frontmatter `status` from `stub` to `active` and sets `last_updated`.
    - **Acceptance criteria**:
      - [ ] Feature spec creation path is `.eng-docs/specs/backlog/feature-[name].md`
      - [ ] Frontmatter is populated at creation: `created`, `last_updated`, `status: draft`, `specced_by`
      - [ ] Stage doc instructs setting wiki `status: active` on first substantive edit to a wiki file
    - **Dependencies**: Task: Add frontmatter to spec templates, Task: Add frontmatter to wiki templates
  - [x] **Task: Update `adrs.md` — create ADR with frontmatter; set `decided_by` at `accepted` transition**
    - **Description**: In `references/stages/adrs.md`, update the ADR creation step to populate frontmatter: `created`, `last_updated`, `status: proposed`, `decided_by: null`, `superseded_by: null`. Add an instruction at the approval checkpoint to set `decided_by` to the current GitHub username and update `status` to `accepted`.
    - **Acceptance criteria**:
      - [ ] ADR creation step populates all frontmatter fields
      - [ ] Approval checkpoint step sets `decided_by` and transitions `status` to `accepted`
      - [ ] `last_updated` is set at both creation and the `accepted` transition
    - **Dependencies**: Task: Add frontmatter to ADR template

- [x] **Story: Update stage docs — lifecycle transitions**
  - [x] **Task: Update `task-decomposition.md` — set `status: approved` at checkpoint**
    - **Description**: In `references/stages/task-decomposition.md`, add an instruction to the human approval checkpoint step: after the human approves the task list, update the spec file's frontmatter `status` from `draft` to `approved` and set `last_updated`.
    - **Acceptance criteria**:
      - [ ] Checkpoint step includes instruction to set `status: approved` in the spec frontmatter
      - [ ] `last_updated` is set at the same time
      - [ ] Instruction specifies editing the spec file at its current path (which may be in `backlog/`)
    - **Dependencies**: Task: Add frontmatter to spec templates
  - [x] **Task: Update `implementation-handoff.md` — move spec from `backlog/`; set `implementing` and `complete`**
    - **Description**: In `references/stages/implementation-handoff.md`, add instructions for three frontmatter operations: (1) when the worktree is created (step 5), move the spec from `.eng-docs/specs/backlog/` to `.eng-docs/specs/` using `git mv`, then set `status: implementing`, `implemented_by` to the current GitHub username, and `last_updated`; (2) after the PR is created (end of step 6), set `status: complete` and `last_updated`.
    - **Acceptance criteria**:
      - [ ] Step 5 includes `git mv .eng-docs/specs/backlog/[name].md .eng-docs/specs/[name].md`
      - [ ] Step 5 sets `status: implementing`, `implemented_by`, and `last_updated` after the move
      - [ ] Post-PR step sets `status: complete` and `last_updated`
      - [ ] Instructions note that if the spec is already in `specs/` (not `backlog/`), skip the `git mv`
    - **Dependencies**: Task: Add frontmatter to spec templates

- [x] **Story: Update skill reference docs**
  - [x] **Task: Update `planning/SKILL.md` artifact structure section**
    - **Description**: In `plugins/caf/skills/planning/SKILL.md`, update the artifact structure section to: (1) add `specs/backlog/` as a subfolder entry with description "unimplemented specs; moved to `specs/` at implementation handoff", and (2) add a note that all artifact files begin with a YAML frontmatter block whose `status` field reflects lifecycle state.
    - **Acceptance criteria**:
      - [ ] Artifact structure section lists `specs/backlog/` with its description
      - [ ] A frontmatter convention note is present in the artifact structure section
    - **Dependencies**: None
  - [x] **Task: Update `friction-log/SKILL.md` — canonicalize frontmatter schema**
    - **Description**: In `plugins/caf/skills/friction-log/SKILL.md`, update the file creation step to explicitly specify the frontmatter schema: `created`, `last_updated`, `status: untriaged`, `captured_by`, `triaged_by: null`. Valid status values are `untriaged | partially-triaged | triaged`. This makes the schema normative rather than implicit from existing files.
    - **Acceptance criteria**:
      - [ ] File creation step specifies the full frontmatter schema with all fields
      - [ ] Valid status values (`untriaged | partially-triaged | triaged`) are documented
      - [ ] `captured_by` is set to the current GitHub username at creation
    - **Dependencies**: None
