# openspec/changes

Feature specs for the deep research pipeline.

## Structure

```
changes/
  <feature-name>/          # active or in-progress features
    proposal.md            # scope, non-goals, context (newer specs)
    design.md              # architecture and integration detail
    tasks.md               # phased implementation tasks
    — or —
    requirements.md        # older specs: scope + constraints
    plan.md                # older specs: implementation steps
    validation.md          # older specs: acceptance criteria
  archived/
    <feature-name>/        # completed and merged features
```

## Active features

| Feature | Format | Status |
|---------|--------|--------|
| `page-level-image-discovery-mcp` | openspec (proposal/design/tasks) | In progress |
| `agent-validation` | legacy (requirements/plan/validation) | In progress |
| `image-fetching` | legacy (requirements/plan/validation) | Not started |

## Archived features

| Feature | Archived date | Notes |
|---------|--------------|-------|
| `langfuse-migration` | 2026-04-24 | All automated gates passed; PR open |
| `trend-skill` | 2026-04-24 | Groups 0–5 complete on development branch |
