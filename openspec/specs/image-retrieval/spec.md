# Spec: Image Retrieval

## ADDED Requirements

### Requirement: Image embedding index construction
系统 SHALL 从所有报告目录的 `images/images_metadata.json` 中加载图片元数据，使用 TEXT-EMBEDDING-3-SMALL 对每张图片的 `description` 字段计算 embedding 向量，并将结果缓存至 `material_library/image_embeddings.npz`。

#### Scenario: First-time index build
- **WHEN** `material_library/image_embeddings.npz` 不存在
- **THEN** 系统 SHALL 扫描所有 `reports/*/images/images_metadata.json`，提取全部图片元数据，batch 计算 description embedding，将 embeddings 矩阵和 metadata 数组保存为 `.npz` 缓存文件

#### Scenario: Cache hit
- **WHEN** `material_library/image_embeddings.npz` 已存在且图片数量与 `images_metadata.json` 总数一致
- **THEN** 系统 SHALL 直接加载缓存的 embedding 矩阵和 metadata，不重新计算

#### Scenario: Cache invalidation
- **WHEN** 缓存文件已存在，但任一 `images_metadata.json` 的修改时间比缓存文件更新，或图片总数与缓存不一致
- **THEN** 系统 SHALL 删除旧缓存并完整重建索引

### Requirement: Element-to-image retrieval strategy
The system SHALL change the retrieval strategy used by `attach_images` from pure embedding search over the full image index to a graph-first plus embedding-rerank flow. The function signature and output format SHALL remain unchanged.

#### Scenario: Retrieval executes for a recommended element
- **WHEN** `attach_images` looks up reference images for a recommended element
- **THEN** the system SHALL first recall graph-linked images for that element's `material_id`, rerank only those candidates with embedding cosine similarity, and return the result in the same `ImageReference` list format as before

#### Scenario: Callers depend on the existing interface
- **WHEN** existing recommender code invokes the image retrieval helper or consumes its results
- **THEN** no caller-visible signature change or output schema change SHALL be required

### Requirement: attach_images graph node
推荐 graph SHALL 在 `recommend` node 之后包含一个 `attach_images` node，该 node 读取 state 中的推荐结果，对每个 `ElementRecommendation` 执行图片检索，将匹配的图片写入 `reference_images` 字段。

#### Scenario: Graph execution order
- **WHEN** 用户调用推荐 agent
- **THEN** graph 的执行顺序 SHALL 为 `START → recommend → attach_images → END`

#### Scenario: Recommendation with images
- **WHEN** `recommend` node 产出包含 N 个推荐元素的结果
- **THEN** `attach_images` node SHALL 为每个元素填充 `reference_images` 字段（每个元素最多 3 张图片），并更新 state 中的 `recommendations`

#### Scenario: No recommendations
- **WHEN** `recommend` node 的结果中某个维度的推荐列表为空
- **THEN** `attach_images` node SHALL 跳过该维度，不报错

### Requirement: ImageReference output schema
每张匹配的参考图片 SHALL 以 `ImageReference` Pydantic model 表示，包含 `local_path`（绝对本地路径）和 `description`（中文描述）。

#### Scenario: ImageReference structure
- **WHEN** 图片被检索匹配到某个推荐元素
- **THEN** 该图片 SHALL 被表示为 `ImageReference(local_path="/home/.../images/xxx.jpg", description="中文描述")`

#### Scenario: Display in conversation
- **WHEN** 推荐结果被格式化为对话消息
- **THEN** 每个元素的参考图片 SHALL 以 `📷 描述 (路径)` 格式显示在推荐理由之后

## MODIFIED Requirements

### Requirement: ElementRecommendation schema
`ElementRecommendation` model SHALL 包含一个新字段 `reference_images: list[ImageReference]`，默认为空列表。该字段由 `attach_images` node 在推荐完成后填充，不由 LLM 生成。

#### Scenario: Schema backward compatibility
- **WHEN** `attach_images` node 未执行（如直接调用 `recommend` node）
- **THEN** `reference_images` SHALL 默认为空列表 `[]`，不影响已有功能
