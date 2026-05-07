# Tasks: page-level-image-discovery-mcp

## Group 1 — MCP Discovery Foundation

- [ ] Define the production-path MCP client strategy for page inspection in notebook 2.
- [ ] Add browser-first and fetch / HTTP fallback configuration for page inspection helpers.
- [ ] Decide whether `ImageResult` needs optional metadata extensions for page-derived context.

## Group 2 — Page Image Extraction

- [ ] Add deterministic helpers to inspect Tavily-selected pages with MCP tools.
- [ ] Extract `img`, `alt`, `figcaption`, `og:image`, and page title data.
- [ ] Normalize relative URLs and page-derived metadata into `ImageResult` records.

## Group 3 — Pipeline Integration

- [ ] Merge MCP-discovered images with Tavily-provided images in the notebook 2 search helper path.
- [ ] Preserve the merged image list through researcher state, compression, and supervisor aggregation.
- [ ] Ensure final report generation continues to download discovered images into the report image directory and can embed them with local relative paths.

## Group 3.5 — Material Metadata Enrichment

- [ ] Define optional `ImageResult` extensions for page context and material metadata fields.
- [ ] Add enrichment helpers that derive structured metadata from page context (`alt`, `figcaption`, nearby text, page title) and fetched image facts.
- [ ] Load and use material library catalogs (`color.json`, `texture.json`, `decoration.json`, `style.json`) as constrained matching targets.
- [ ] Link each image to candidate material library entries with confidence and evidence (`dimension`, target id/name, evidence source/text).
- [ ] Persist material-aware metadata and links in `images_metadata.json` alongside existing fields.
- [ ] Add selection/ranking rules so report embedding can prefer images with stronger material-library links when output must be bounded.

## Group 4 — Validation

- [ ] Add automated tests for normalization, fallback behavior, and duplicate handling.
- [ ] Add automated coverage for preserving page-derived metadata into the report/download path.
- [ ] Add automated tests for image-to-material-library linking across color, texture, decoration, and style.
- [ ] Add automated tests for confidence/evidence persistence and no-match fallback behavior.
- [ ] Run lint and targeted validation on the affected pipeline.

## Group 5 — Documentation and Review

- [ ] Update feature specs or roadmap notes if implementation scope changes from the current assumptions.
- [ ] Review notebook/source-of-truth boundaries before merge.
- [ ] Prepare implementation follow-up via `/opsx:apply`.
