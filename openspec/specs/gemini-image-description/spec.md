# Spec: AI-Generated Image Description via Gemini

## Summary

After the research pipeline downloads images, the `describe_images_with_gemini()` function calls Gemini 2.5 flash (vision → text) for each successfully downloaded image to produce a concise natural-language description emphasising juice visual properties (color, texture, opacity, decorations, glass/packaging style).

---

## Goal

Each downloaded image in `images_metadata.json` carries a meaningful AI-generated description so the image library is semantically searchable by visual characteristics.

---

## Scope

- New function `describe_images_with_gemini(images: list[ImageResult]) -> list[ImageResult]` in `utils.py`.
- Reads the image file at `local_path`, base64-encodes it, and sends a single multimodal request to the `NANO_BANANA_FLASH_URL` endpoint (Gemini 2.5 flash `generateContent`).
- The description SHALL mention juice color, texture, opacity, and decorations (fruit garnish, ice, glass style, packaging) when these are visible.
- Calls are sequential (one image at a time).
- Uses `GenAIToken` for Azure AD token-based auth and the existing `httpx` client pattern.
- Graceful degradation: if the API call fails or the env var is unset, the function logs a warning and returns descriptions as `""` without raising.

---

## Non-Goals

- Concurrency / batching of Gemini calls (deferred to a future optimisation).
- Retrying failed Gemini calls.
- Structured or JSON output from Gemini — plain text is sufficient.
- Changing the `ImageResult` Pydantic model.
- Modifying LangGraph state, report rendering, or any other pipeline stage.

---

## Constraints

1. **Notebooks are source of truth.** All changes are in `%%writefile` cells in `notebooks/utils.py`; never edit `src/` directly.
2. **Env var**: `NANO_BANANA_FLASH_URL` must point to the `gemini-2.5-flash:generateContent` endpoint. The function does nothing (logs a warning, returns input unchanged) when the var is unset or empty.
3. **Auth**: uses `GenAIToken` with `HEADERS_USERID` and `HEADERS_PROJECT_NAME` — same pattern as existing Gemini image-gen calls.
4. **Failure isolation**: a single-image failure must not abort the loop or raise to the caller.

---

## Requirements

### Requirement: Generate AI description for a downloaded image

The system SHALL call Gemini 2.5 flash (vision → text) for each image that has a `local_path`, reading the image file and producing a concise English description. The description SHALL mention juice color, texture, opacity, and decorations (e.g., fruit garnish, ice, glass style, packaging) when these are visible in the image.

#### Scenario: Image shows juice product
- **WHEN** `describe_images_with_gemini()` is called with an image whose `local_path` points to a file showing a juice product
- **THEN** the returned description includes at least one visually distinctive property (color, texture, opacity, or decoration) of the juice

#### Scenario: Image does not show juice
- **WHEN** `describe_images_with_gemini()` is called with an image that does not contain juice
- **THEN** the returned description describes what is visible without fabricating juice properties

#### Scenario: Gemini call fails (network or API error)
- **WHEN** the Gemini API call raises an exception or returns a non-2xx status
- **THEN** the system logs a warning and returns an empty string for `description`; it SHALL NOT raise an exception

#### Scenario: `NANO_BANANA_FLASH_URL` is not configured
- **WHEN** `describe_images_with_gemini()` is called and `NANO_BANANA_FLASH_URL` env var is unset or empty
- **THEN** the system logs a warning, skips all Gemini calls, and returns the input list unchanged (all descriptions remain `""`)

---

## Acceptance Criteria

1. `describe_images_with_gemini()` exists in `utils.py` and accepts a `list[ImageResult]`.
2. For each image with a non-null `local_path`, the function makes exactly one Gemini API call.
3. On success, the `description` field of the returned `ImageResult` is a non-empty string.
4. On API failure, the `description` field is `""` and no exception propagates.
5. When `NANO_BANANA_FLASH_URL` is unset, the function returns the input list unchanged with no API calls made.
6. `ruff check src/` passes with no new errors after the notebook cell is re-run.

---

## Validation Plan

1. Set `NANO_BANANA_FLASH_URL` and call `describe_images_with_gemini()` with one downloaded juice image — verify `description` is non-empty and mentions a visual property.
2. Call with a non-juice image — verify description does not fabricate juice content.
3. Simulate API failure (wrong URL) — verify `description` is `""` and no exception is raised.
4. Unset `NANO_BANANA_FLASH_URL` — verify all descriptions remain `""` and a warning is logged.
5. Run `ruff check src/` and confirm no new lint errors.

---

## Risks / Open Questions

**R1. Latency**: ~100 sequential Gemini calls add ~3–5 min to the pipeline. Acceptable for now; concurrency can be added later.

**R2. Prompt hallucination**: Gemini may describe non-juice images as if they contain juice. Mitigated by prompting Gemini to describe juice properties *only if visible*.

**R3. Quota**: Each call consumes Gemini quota. No mitigation needed at current usage scale.
