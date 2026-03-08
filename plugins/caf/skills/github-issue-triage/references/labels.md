# Label Taxonomy

Use these exact names, colors, and descriptions when creating labels with `gh label create`.

## Type labels

| Label | Color | Description |
|-------|-------|-------------|
| `bug` | `#d73a4a` | Something isn't working correctly |
| `feature-request` | `#a2eeef` | A request for new functionality |
| `enhancement` | `#84b6eb` | Improvement to existing functionality |
| `documentation` | `#0075ca` | Improvements or additions to documentation |
| `question` | `#d876e3` | Further information is requested |

## Priority labels

| Label | Color | Description |
|-------|-------|-------------|
| `P0: critical` | `#b60205` | Urgent — production broken, data loss, or security issue |
| `P1: high` | `#e4460a` | Significant user-facing breakage with no workaround |
| `P2: medium` | `#e4b429` | Bug with workaround, or high-value feature request |
| `P3: low` | `#fef2c0` | Minor issue, polish, or nice-to-have |

## Meta labels

| Label | Color | Description |
|-------|-------|-------------|
| `needs-info` | `#cccccc` | More information needed from the reporter before this can be triaged |
| `usability` | `#f9d0c4` | Relates to user experience or interface usability |
| `performance` | `#c5def5` | Relates to speed, memory, or resource usage |
| `security` | `#ee0701` | Has security implications — handle with care |
| `good-first-issue` | `#7057ff` | Good entry point for new contributors |

## Creating labels

```bash
# Type labels
gh label create "bug" --color "d73a4a" --description "Something isn't working correctly"
gh label create "feature-request" --color "a2eeef" --description "A request for new functionality"
gh label create "enhancement" --color "84b6eb" --description "Improvement to existing functionality"
gh label create "documentation" --color "0075ca" --description "Improvements or additions to documentation"
gh label create "question" --color "d876e3" --description "Further information is requested"

# Priority labels
gh label create "P0: critical" --color "b60205" --description "Urgent — production broken, data loss, or security issue"
gh label create "P1: high" --color "e4460a" --description "Significant user-facing breakage with no workaround"
gh label create "P2: medium" --color "e4b429" --description "Bug with workaround, or high-value feature request"
gh label create "P3: low" --color "fef2c0" --description "Minor issue, polish, or nice-to-have"

# Meta labels
gh label create "needs-info" --color "cccccc" --description "More information needed from the reporter before this can be triaged"
gh label create "usability" --color "f9d0c4" --description "Relates to user experience or interface usability"
gh label create "performance" --color "c5def5" --description "Relates to speed, memory, or resource usage"
gh label create "security" --color "ee0701" --description "Has security implications — handle with care"
gh label create "good-first-issue" --color "7057ff" --description "Good entry point for new contributors"
```

## Additional labels

| Label | Color | Description |
|-------|-------|-------------|
| `regression` | `#e4460a` | Previously working behavior that broke |
| `accessibility` | `#bfd4f2` | Affects users with disabilities or assistive technology needs |
| `flaky-test` | `#fef2c0` | Test that fails intermittently without code changes |
| `api` | `#c2e0c6` | Relates to an external or internal API contract |
| `breaking-change` | `#b60205` | Change that is not backward-compatible |

### Creating additional labels

```bash
gh label create "regression" --color "e4460a" --description "Previously working behavior that broke"
gh label create "accessibility" --color "bfd4f2" --description "Affects users with disabilities or assistive technology needs"
gh label create "flaky-test" --color "fef2c0" --description "Test that fails intermittently without code changes"
gh label create "api" --color "c2e0c6" --description "Relates to an external or internal API contract"
gh label create "breaking-change" --color "b60205" --description "Change that is not backward-compatible"
```
