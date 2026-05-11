## 1. Remove httpx Page Discovery Helpers from utils.py (Notebook 2 Cell 7)

- [x] 1.1 Remove `normalize_page_image()` function
- [x] 1.2 Remove `discover_page_images()` function
- [x] 1.3 Remove `batch_discover_images()` function
- [x] 1.4 Remove `merge_image_lists()` function
- [x] 1.5 Remove `reset_page_discovery_cache()` function
- [x] 1.6 Remove `_get_inspected_urls()` helper function

## 2. Simplify threading.local Machinery in utils.py

- [x] 2.1 Remove `_thread_local = threading.local()` module-level variable
- [x] 2.2 Replace it with a simple module-level `_last_search_images: list[ImageResult] = []`
- [x] 2.3 Rewrite `get_last_search_images()` to read from the module-level list (no threading.local)
- [x] 2.4 Update `tavily_search` tool: remove `batch_discover_images` / `merge_image_lists` calls; store `extract_images_from_search_results()` result directly into `_last_search_images`

## 3. Clean Up Unused Imports in utils.py

- [x] 3.1 Remove `import threading`
- [x] 3.2 Remove `from bs4 import BeautifulSoup`
- [x] 3.3 Remove `from concurrent.futures import ThreadPoolExecutor, as_completed`
- [x] 3.4 Verify no other usages remain for these imports before removing

## 4. Regenerate Source Files and Clean Up Tests

- [x] 4.1 Run the `%%writefile` cells in notebook 2 to regenerate `src/deep_research_from_scratch/utils.py` and `state_research.py`
- [x] 4.2 Delete `tests/test_page_image_discovery.py` (all tests cover removed code)
- [x] 4.3 Run remaining tests to verify nothing is broken: `uv run pytest tests/ -v`
- [x] 4.4 Run `ruff check src/` and fix any lint issues in the notebook cells

## 5. Archive Prior Change and Commit

- [x] 5.1 Archive `page-level-image-discovery-mcp`: `openspec archive --change "page-level-image-discovery-mcp"`
- [x] 5.2 Commit all changes to `development` branch with a clear message
