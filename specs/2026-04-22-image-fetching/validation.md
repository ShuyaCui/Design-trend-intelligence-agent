# Validation: Research Agent Image Fetching

## Success Criteria

The feature is considered complete and ready to merge when **all** of the following are verified:

---

## Functional Validation

### V1 — Image URL Collection
- [ ] `tavily_search` tool output includes image URLs when Tavily returns images
- [ ] `ImageResult` objects are correctly constructed with `url`, `title`, `source_page` fields
- [ ] When Tavily returns no images, the tool output is unchanged from current behavior

### V2 — Image Download
- [ ] Images are downloaded to `reports/<session_id>/images/` directory
- [ ] Download failures (timeout, 404, invalid format) are logged but do not block research
- [ ] `images_metadata.json` is written with correct metadata for all downloaded images
- [ ] `DISABLE_SSL_VERIFY` setting is respected during image downloads

### V3 — State Flow
- [ ] `images` field accumulates correctly across multiple tool call iterations in `ResearcherState`
- [ ] Images from multiple parallel sub-agents are aggregated in `SupervisorState`
- [ ] Images flow from `SupervisorState` through to `AgentState` in the full pipeline

### V4 — Compressed Research
- [ ] `compress_research` output includes references to relevant images
- [ ] Image references survive compression (not dropped by summarization)

### V5 — Final Report
- [ ] Final report contains `![caption](path)` Markdown image references
- [ ] Image paths in report are valid relative paths to downloaded files
- [ ] Report is still well-structured when no images are available

---

## Backward Compatibility

### V6 — No Regression
- [ ] Existing text-only research flow works without errors when no images are returned
- [ ] Empty `images: []` default does not cause state errors
- [ ] All existing LangGraph entry points still compile (`research_agent_full.py:agent`, `research_agent.py:researcher_agent`, `multi_agent_supervisor.py:supervisor_agent`)

---

## Code Quality

### V7 — Lint & Style
- [ ] `ruff check src/` passes with no errors
- [ ] All new code follows existing conventions (Google-style docstrings, import ordering)

### V8 — Notebook Workflow
- [ ] Modified notebook cells (2, 4, 5) regenerate `src/` files correctly via `%%writefile`
- [ ] No direct edits to `src/` files exist outside of `%%writefile` generation

---

## Merge Checklist

Before merging `development` → `main`:

1. All V1–V8 checks pass
2. Feature spec files exist in `specs/2026-04-22-image-fetching/`
3. `specs/roadmap.md` updated with image fetching phase
4. `specs/tech-stack.md` updated with image capabilities
5. Changes committed to `development` branch with descriptive commit message
6. PR opened from `development` → `main` for human review
