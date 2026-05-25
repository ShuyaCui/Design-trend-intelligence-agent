## ADDED Requirements

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

### Requirement: Zero-result handling from the KG
The system SHALL return an empty image list when graph recall returns no images for a material, and SHALL NOT fall back to embedding-only retrieval over the full index.

#### Scenario: No graph-linked images are found
- **WHEN** `get_images_for_material()` returns zero image paths for a material
- **THEN** the system SHALL return an empty list for that element's reference images and SHALL NOT perform a full-index embedding search

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
