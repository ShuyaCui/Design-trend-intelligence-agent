# Spec: Image Download with AI-Enriched Metadata

## Summary

`download_images()` in `utils.py` downloads a list of `ImageResult` objects into `reports/<session_id>/images/`, then immediately calls `describe_images_with_gemini()` to enrich successfully downloaded images with AI-generated descriptions, and finally writes a filtered `images_metadata.json` containing only downloaded images with exactly three fields: `url`, `local_path`, and `description`.

---

## Goal

The `images_metadata.json` produced after each research session contains only images that were actually downloaded, each paired with an AI-generated description, so the file is semantically useful for property-based retrieval without noise from failed downloads or empty metadata fields.

---

## Scope

- `download_images(images: list[ImageResult], output_dir: Path) -> list[ImageResult]` in `utils.py`.
- Downloads each image to `output_dir/images/<filename>` (best-effort; failures are logged but do not abort the loop).
- After the download loop completes, calls `describe_images_with_gemini()` on the list of successfully downloaded images.
- Writes `images_metadata.json` to `output_dir/` containing only images where `local_path` is not null, serialised with only the fields `url`, `local_path`, and `description`.
- All changes are in `notebooks/utils.py` via `%%writefile`; `src/` is regenerated.

---

## Non-Goals

- Retrying or re-downloading failed images.
- Changing the `ImageResult` Pydantic model (stays rich for in-pipeline use).
- Modifying any LangGraph state, report rendering, or supervisor logic.
- Structured output from Gemini (handled by `gemini-image-description` spec).

---

## Constraints

1. **Notebooks are source of truth.** All changes in `%%writefile` cells; never edit `src/` directly.
2. **Best-effort downloads.** A download failure must log a warning and continue; it must not raise or abort the batch.
3. **Description enrichment before metadata write.** `describe_images_with_gemini()` must be called before `images_metadata.json` is written so descriptions are present in the file.
4. **Filtered output.** Only images with a non-null `local_path` appear in `images_metadata.json`.

---

## Requirements

### Requirement: Write filtered images_metadata.json after download

After all download attempts complete, `download_images()` SHALL write `images_metadata.json` containing ONLY images that were successfully downloaded (i.e., `local_path` is not null), and ONLY the fields `url`, `local_path`, and `description` per entry.

#### Scenario: Mix of successful and failed downloads
- **WHEN** `download_images()` is called with a list that includes both successful and failed downloads
- **THEN** `images_metadata.json` contains only entries for successfully downloaded images, each with exactly three fields: `url`, `local_path`, `description`

#### Scenario: All downloads fail
- **WHEN** every image in the input list fails to download
- **THEN** `images_metadata.json` is written as an empty JSON array `[]`

#### Scenario: Description enrichment completes before metadata write
- **WHEN** `download_images()` completes the download loop
- **THEN** `describe_images_with_gemini()` is called before `images_metadata.json` is written, so descriptions are present in the file

### Requirement: Call description enrichment as final step in download_images

`download_images()` SHALL call `describe_images_with_gemini()` on the list of successfully downloaded images immediately after the download loop, before writing `images_metadata.json`.

#### Scenario: Enrichment integrates into download pipeline
- **WHEN** `download_images()` is called
- **THEN** the `images_metadata.json` output contains AI-generated descriptions for images where `NANO_BANANA_FLASH_URL` is configured and calls succeed

---

## Acceptance Criteria

1. `download_images()` calls `describe_images_with_gemini()` after the download loop and before writing `images_metadata.json`.
2. `images_metadata.json` contains only entries where `local_path` is not null.
3. Each entry in `images_metadata.json` has exactly three fields: `url`, `local_path`, `description`.
4. When all downloads fail, `images_metadata.json` is written as `[]`.
5. Download failures are logged and do not raise exceptions or abort the batch.
6. `ruff check src/` passes with no new errors after notebook cells are re-run.

---

## Validation Plan

1. Run `download_images()` with a mix of valid and invalid URLs; verify `images_metadata.json` excludes failed entries.
2. Run `download_images()` where all URLs are invalid; verify `images_metadata.json` is `[]`.
3. Run with `NANO_BANANA_FLASH_URL` set; verify `images_metadata.json` entries have non-empty `description` values.
4. Run with `NANO_BANANA_FLASH_URL` unset; verify `images_metadata.json` entries have `description: ""`.
5. Confirm `images_metadata.json` contains exactly three keys per entry: `url`, `local_path`, `description`.
6. Run `ruff check src/` and confirm no new lint errors.

---

## Risks / Open Questions

**R1. Latency**: Gemini enrichment (sequential, ~2 s/image) adds ~3–5 min for ~100 images. Acceptable for now; concurrency is a future optimisation.

**R2. Schema change**: `images_metadata.json` now has fewer fields and excludes failed downloads. Downstream consumers that relied on the old schema (8 fields, failed entries included) will need to be updated if any exist.
