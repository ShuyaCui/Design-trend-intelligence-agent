## 1. Phase A — Schema & Skeleton

- [x] 1.1 Define Pydantic models (`MaterialElement`, `DimensionFile`, `PersonaCatalog`, `IndexMetadata`) in `scripts/material_schema.py`
- [x] 1.2 Create `material_library/` directory and empty `index.json` with initial structure
- [x] 1.3 Define the 6 predefined aesthetic personas with descriptions and example cross-dimension combinations in a `PERSONA_CATALOG` constant

## 2. Phase B — Extraction Core

- [x] 2.1 Write the LLM extraction prompt that instructs the model to parse a Markdown report and output `ReportExtraction` JSON (covering 颜色, 装饰物, 透明度与质地)
- [x] 2.2 Implement `extract_single_report(report_path) -> ReportExtraction` function using `init_chat_model` + `with_structured_output(ReportExtraction)`
- [x] 2.3 Implement `extract_all_reports(reports_dir, force=False)` with incremental logic: skip already-processed reports (check `index.json`), process new ones, collect all elements
- [x] 2.4 Implement `build_dimension_files(elements, output_dir)` that splits all elements by dimension, groups by maturity, and writes `color.json`, `decoration.json`, `texture.json`
- [x] 2.5 Implement `build_personas_file(elements, output_dir)` that generates `personas.json` with the persona catalog and cross-dimension combinability examples
- [x] 2.6 Implement `update_index(output_dir, processed_reports)` to write/update `index.json` with processing metadata and per-dimension element counts

## 3. Phase B — CLI Entry Point

- [x] 3.1 Create `scripts/extract_material_library.py` with `argparse` CLI: `--reports-dir`, `--output-dir`, `--force` flags
- [x] 3.2 Wire CLI to call `extract_all_reports()` then `build_dimension_files()` then `build_personas_file()` then `update_index()`
- [x] 3.3 Run extraction on all 3 existing reports; verify `color.json`, `decoration.json`, `texture.json`, `personas.json` output structure and element completeness

## 4. Phase C — Validation & Quality

- [x] 4.1 Spot-check: compare extracted element count per dimension against report chapter count (颜色趋势 1-6, 装饰趋势 7-12, etc.) to ensure no elements were missed
- [x] 4.2 Validate all elements pass Pydantic schema validation (no missing fields, correct enum values)
- [x] 4.3 Validate `personas.json`: all personas present; all elements' `aesthetic_persona` references a valid persona
- [x] 4.4 Validate dimension files: elements correctly split across `color.json`, `decoration.json`, `texture.json`; maturity grouping is correct
- [x] 4.4 Ruff lint check on `scripts/material_schema.py` and `scripts/extract_material_library.py`

## 5. Phase C — Documentation & Commit

- [x] 5.1 Add a README section in `material_library/README.md` explaining the schema, directory structure, and how to run the extraction
- [x] 5.2 Commit all changes to `development` branch with summary of files changed
