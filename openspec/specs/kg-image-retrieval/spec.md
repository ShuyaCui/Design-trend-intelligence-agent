## ADDED Requirements

### Requirement: 按素材检索代表图片
系统 SHALL 提供 Python 函数，给定素材 id 或素材名称，返回所有与该素材相连的图片列表。

#### Scenario: 按 material_id 检索
- **WHEN** 调用 `get_images_for_material(material_id="25fccecd-color-90b96051-26da05")`
- **THEN** 系统 SHALL 执行 Cypher `MATCH (i:Image)-[]->(m:Material {id: $id}) RETURN i` 并返回图片节点属性列表（含 `path`、`description`、`report_id`、`filename`）

#### Scenario: 按 material_name 检索
- **WHEN** 调用 `get_images_for_material(material_name="低饱和香氛色")`
- **THEN** 系统 SHALL 执行按 `name` 属性的匹配查询，返回相同格式的图片列表

#### Scenario: 无匹配时返回空列表
- **WHEN** 给定素材无任何图片与之相连
- **THEN** 函数 SHALL 返回空列表 `[]`，不抛出异常

### Requirement: 按维度过滤检索
系统 SHALL 支持仅通过特定关系类型（颜色/质地/装饰）检索图片。

#### Scenario: 仅查颜色关系
- **WHEN** 调用 `get_images_for_material(material_id="...", relation="HAS_COLOR")`
- **THEN** 系统 SHALL 仅返回通过 `[:HAS_COLOR]` 关系连接的图片

#### Scenario: 查全部关系（默认）
- **WHEN** 调用 `get_images_for_material(material_id="...")` 不指定 relation
- **THEN** 系统 SHALL 返回通过任意关系（HAS_COLOR、HAS_TEXTURE、HAS_DECORATION）连接的图片

### Requirement: 按报告过滤
系统 SHALL 支持按 report_id 过滤检索结果。

#### Scenario: 按 report 过滤
- **WHEN** 调用 `get_images_for_material(material_id="...", report_id="e602b08d-...")`
- **THEN** 系统 SHALL 仅返回属于该报告的图片

### Requirement: 素材统计查询
系统 SHALL 提供统计接口，支持查询各素材的代表图片数量。

#### Scenario: 素材图片计数
- **WHEN** 调用 `get_material_image_counts(dimension="颜色")`
- **THEN** 系统 SHALL 返回该维度所有素材的 `{material_name: image_count}` 字典，按 count 降序排列

#### Scenario: 全维度统计
- **WHEN** 调用 `get_material_image_counts()` 不指定维度
- **THEN** 系统 SHALL 返回全部 107 个素材的图片计数

### Requirement: Graph recall for material images
The system SHALL retrieve candidate images for a recommended material by calling `get_images_for_material(material_id)` and using the returned image paths as the recall set for that material.

#### Scenario: Material has linked images in the KG
- **WHEN** `attach_images` processes a recommendation with `element_id`
- **THEN** the system SHALL call `get_images_for_material(material_id=element_id)` and use all returned image paths as the candidate image set for reranking

### Requirement: Embedding rerank over graph-filtered candidates
The system SHALL compute cosine similarity between the query embedding and the subset of cached image embeddings whose metadata paths match the images returned from the KG.

#### Scenario: Graph recall returns candidate images
- **WHEN** one or more image paths are returned by `get_images_for_material()`
- **THEN** the system SHALL filter `image_embeddings.npz` metadata and embedding rows to only those paths, compute cosine similarity against the query embedding, and rank the candidates by similarity score

### Requirement: Query construction for reranking
The system SHALL construct the image reranking query using the same format as the current retrieval flow: `element_name + visual_keywords`.

#### Scenario: Element has visual keywords
- **WHEN** a recommended element includes `element_name` and one or more `visual_keywords`
- **THEN** the system SHALL construct the query as the element name followed by the visual keywords in the same format used by the existing retrieval logic

#### Scenario: Element has no visual keywords
- **WHEN** a recommended element includes `element_name` and an empty `visual_keywords` list
- **THEN** the system SHALL use `element_name` as the full query string

### Requirement: Top-N image selection
The system SHALL return the top reranked images by similarity score, with a default of 3 images and support for a configurable top-N value.

#### Scenario: More than N graph-linked images are available
- **WHEN** the reranked candidate set contains more matches than the configured `top_k`
- **THEN** the system SHALL return only the highest-scoring `top_k` images, defaulting to 3

### Requirement: Configurable retrieval mode
The system SHALL support an `image_retrieval_mode` configuration parameter read from LangGraph's `configurable` field. Valid values are `"kg_rerank"` (default) and `"embedding_only"`.

#### Scenario: Default mode
- **WHEN** `image_retrieval_mode` is not explicitly set
- **THEN** the system SHALL use `"kg_rerank"` as the retrieval strategy

#### Scenario: Embedding-only mode selected
- **WHEN** `image_retrieval_mode` is set to `"embedding_only"`
- **THEN** the system SHALL use the existing full-index embedding search over all 275 images, preserving current behavior

### Requirement: Zero-result handling from the KG
The system SHALL return an empty image list when graph recall returns no images for a material in `"kg_rerank"` mode, and SHALL NOT fall back to full-index embedding retrieval.

#### Scenario: No graph-linked images are found (kg_rerank mode)
- **WHEN** `get_images_for_material()` returns zero image paths for a material
- **THEN** the system SHALL return an empty list for that element's reference images

### Requirement: ImageReference output format
The system SHALL keep the current output contract by returning matched images as `ImageReference` objects containing `local_path` and `description`.

#### Scenario: Reranked images are returned
- **WHEN** the graph-plus-rerank retrieval flow selects one or more images
- **THEN** the system SHALL return them as `ImageReference` objects with the same `local_path` and `description` fields used by the current implementation

### Requirement: Neo4j driver availability during image attachment
The recommender flow SHALL ensure a Neo4j driver is available via `get_neo4j_driver()` when the `attach_images` node runs.

#### Scenario: attach_images executes in the recommender graph
- **WHEN** the `attach_images` node starts processing recommendations
- **THEN** a Neo4j driver obtained via `get_neo4j_driver()` SHALL be available to support `get_images_for_material()` calls
