# Feature: Research Agent Image Fetching

## Objective

Extend the research agent to collect relevant images during the research phase, download them to local storage, and include image references in the final report. This transforms the research pipeline from text-only to multimodal (text + images).

## User / Problem Context

Enterprise research reports are more useful when they include relevant charts, diagrams, product screenshots, and data visualizations. Currently, the research agent only captures text content from web searches. Users must manually find and attach images to supplement reports — defeating the purpose of an autonomous research system.

## Scope

### In Scope

- Capture image URLs via Tavily's `include_images=True` parameter during existing web searches
- Download images to local disk at `reports/<session_id>/images/`
- New `ImageResult` Pydantic schema for structured image metadata (URL, title, source page, description)
- Add `images: list[ImageResult]` field to `ResearcherState`, `SupervisorState`, and `AgentState`
- Update `compress_research` node to preserve image references in compressed notes
- Update `final_report_generation` to embed images in the final report using Markdown image syntax
- Update prompts (`research_agent_prompt`, `compress_research_system_prompt`) to instruct agents to record image relevance
- Modify notebooks 2, 4, 5 `%%writefile` cells to regenerate source files

### Non-Goals

- Vision model analysis of image content (future feature)
- Image editing, cropping, or format conversion
- Video content fetching
- Image deduplication beyond URL-level
- New image search API (use Tavily's existing capability only)
- Image thumbnail generation or resizing
- Image content-based relevance scoring

## Key Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Image source | Tavily `include_images` parameter | Zero new dependencies; already part of the search call |
| Storage format | Downloaded image files + `images_metadata.json` | Offline access; report portability |
| Storage location | `reports/{session_id}/images/` | Consistent with existing report output structure |
| State field | `images: Annotated[list[ImageResult], operator.add]` | Accumulates across research iterations via reducer |
| Prompt changes | Instruct agent to note which images are relevant | LLM decides image relevance, not heuristic |
| Report integration | `![caption](relative_path)` in Markdown report | Standard Markdown; works in all renderers |
| Backward compatibility | Empty `images` list is default; no behavior change when no images found | Existing workflows unaffected |
| Download strategy | Best-effort with timeout; failures logged but don't block research | Robustness over completeness |

## Constraints

- **Notebook workflow**: All code changes go through `%%writefile` cells in notebooks, not direct `src/` edits
- **Tavily rate limits**: Image URLs come from existing search calls; no additional API calls needed
- **Disk space**: No explicit limit on image count per session (user responsibility)
- **Network**: Image downloads may fail behind corporate proxies; respect existing SSL config (`DISABLE_SSL_VERIFY`)
- **File formats**: Accept common web image formats (JPEG, PNG, GIF, WebP, SVG); skip unknown formats

## Inputs and Outputs

### Inputs (unchanged)
- User research query → scoping → research brief

### Outputs (enhanced)
- `compressed_research`: Now includes image references
- `images`: List of `ImageResult` objects with metadata
- `reports/<session_id>/images/`: Downloaded image files
- `reports/<session_id>/images_metadata.json`: Structured image metadata
- `final_report`: Markdown report with embedded `![caption](path)` image references

## Open Questions

1. Should there be a configurable max image count per research topic? (Proposed: not in v1, defer to user feedback)
2. Should image captions be LLM-generated or use the source page title? (Proposed: use source page title in v1)
