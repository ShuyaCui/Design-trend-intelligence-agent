## 1. Schema 版本升级

- [ ] 1.1 在 `material_schema.py` 中将 `EXTRACTION_SCHEMA_VERSION` 从 `2` 改为 `3`，触发旧缓存自动失效
- [ ] 1.2 验证：`python -c "from material_schema import EXTRACTION_SCHEMA_VERSION; assert EXTRACTION_SCHEMA_VERSION == 3"`

## 2. 实现 `_chunk_report()` 函数

- [ ] 2.1 在 `extract_material_library.py` 中实现 `_chunk_report(report_text: str, min_chars: int = 200) -> list[str]`：按 H2/H3 标题切分，跳过字数 < min_chars 的 chunk，chunk 数 < 2 时返回 `[report_text]`（降级）
- [ ] 2.2 每个 chunk 前拼接报告前 300 字（preamble）：`preamble + "\n\n---\n\n" + chunk_text`
- [ ] 2.3 单元测试（无 LLM）：构造包含 3 个 H2 章节的模拟报告，验证 `_chunk_report()` 返回 3 个元素，且每个元素以对应标题开头

## 3. 修改 Pass 1 为逐 chunk 提取

- [ ] 3.1 在 `extract_single_report()` 中，将 Pass 1 从单次全文调用改为 `for chunk in _chunk_report(report_text)` 循环，每次调用 `_call_with_retry(three_dim_model, [HumanMessage(content=prompt)])`
- [ ] 3.2 记录每个 chunk 的提取日志（chunk 序号、字数、提取元素数）
- [ ] 3.3 实现 Pass 2 的 H2 切分：单独实现 `_chunk_report_h2only()` 或复用 `_chunk_report()` 的 H2-only 模式，对每个 H2 chunk 调用 `_STYLE_EXTRACTION_PROMPT`，汇总结果后交给 `_deduplicate_elements()`

## 4. 验证与集成测试

- [ ] 4.1 对现有 3 份报告运行 dry-run（mock LLM，验证 chunk 数量是否合理）：`python -c "from extract_material_library import _chunk_report; from pathlib import Path; [print(p.stem[:8], len(_chunk_report(p.read_text())), 'chunks') for p in Path('reports').glob('*/report.md')]"`
- [ ] 4.2 确认 `ruff check src/material-library-extraction/` 通过，无新增 lint 错误
- [ ] 4.3 （可选，需 LLM 环境）运行完整提取，验证每份报告每维度素材数 ≥ 5 条

## 5. 提交

- [ ] 5.1 将变更提交到 `development` 分支，commit message 说明：新增 `_chunk_report()`、Pass 1 改为 chunk 循环、schema version 升为 3
