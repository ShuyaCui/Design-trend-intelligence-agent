## ADDED Requirements

### Requirement: Report is split into section chunks by heading

The system SHALL split a Markdown trend report into a list of section chunks by detecting H2 (`##`) and H3 (`###`) headings. Each chunk SHALL contain the heading line plus all text until the next heading of equal or higher level. If fewer than 2 headings are detected, the system SHALL return the full report as a single chunk (graceful degradation).

#### Scenario: Normal report with multiple H2/H3 headings

- **WHEN** a report contains 5 or more H2/H3 headings
- **THEN** `_chunk_report()` returns a list of strings, one per detected section, each starting with its heading line

#### Scenario: Report with fewer than 2 headings (graceful degradation)

- **WHEN** a report contains 0 or 1 headings
- **THEN** `_chunk_report()` returns a list containing the full report text as a single element

#### Scenario: Short chunks are skipped

- **WHEN** a section chunk contains fewer than 200 characters (after stripping whitespace)
- **THEN** that chunk is excluded from the returned list

### Requirement: Each chunk includes a report preamble for context

The system SHALL prepend the first 300 characters of the full report (the preamble) to each chunk before it is sent to the LLM. This preserves global context (product category, date range, scope) within each focused chunk call.

#### Scenario: Chunk sent to LLM includes preamble

- **WHEN** a chunk is prepared for a Pass 1 LLM call
- **THEN** the content passed to the prompt is `preamble + "\n\n---\n\n" + chunk_text`

### Requirement: Pass 1 extraction iterates over chunks

The system SHALL call the `_THREE_DIM_EXTRACTION_PROMPT` LLM once per chunk (not once per full report). All elements from all chunk calls SHALL be accumulated into a single list before deduplication.

#### Scenario: Multiple chunks produce independent element lists

- **WHEN** a report is split into N chunks
- **THEN** Pass 1 makes exactly N LLM calls, and all returned elements are merged into one list

#### Scenario: Pass 1 skips elements with unexpected dimension

- **WHEN** a chunk extraction returns an element whose `dimension` is not in `{颜色, 装饰物, 透明度与质地}`
- **THEN** that element is discarded with a warning log

### Requirement: Pass 2 (style) is split by H2 headings and extracted per H2 chunk

The system SHALL split the report by H2 (`##`) headings only (not H3) to produce style chunks, then call `_STYLE_EXTRACTION_PROMPT` once per H2 chunk. Style MUST NOT be extracted from the full report in a single call, and MUST NOT use the finer H2/H3 chunks used by Pass 1. All style elements from all H2 chunks SHALL be accumulated and passed through `_deduplicate_elements()` before final output.

#### Scenario: Style extraction uses H2 chunks, not full report

- **WHEN** `extract_single_report()` is called for a report with N H2 sections
- **THEN** exactly N Pass 2 LLM calls are made, one per H2 chunk

#### Scenario: Style H2 chunks include preamble for context

- **WHEN** a style H2 chunk is prepared for a Pass 2 LLM call
- **THEN** the content passed to the prompt is `preamble + "\n\n---\n\n" + h2_chunk_text`

#### Scenario: Style extraction falls back to full report when no H2 found

- **WHEN** the report contains 0 H2 headings
- **THEN** a single Pass 2 LLM call is made with the full report text

#### Scenario: Duplicate style elements across H2 chunks are merged

- **WHEN** the same style name (e.g. "科技净澈") appears in multiple H2 chunks
- **THEN** `_deduplicate_elements()` merges them into one entry with the highest maturity and union of signals/keywords

### Requirement: Schema version is bumped to invalidate v2 caches

`EXTRACTION_SCHEMA_VERSION` SHALL be set to `3`. Any cached `ReportExtraction` with `schema_version != 3` SHALL be re-extracted on the next run.

#### Scenario: Stale v2 cache triggers re-extraction

- **WHEN** a `.cache/*.json` file has `schema_version` of 1 or 2 (or missing)
- **THEN** the pipeline skips the cache and runs fresh extraction for that report
