## Tasks

### 1. Integrate Neo4j driver into recommender graph
- [ ] 1.1 Add Neo4j driver initialization to recommender notebook/state (driver available in attach_images)
- [ ] 1.2 Import kg_retrieval module in material_recommender.py

### 2. Implement graph+rerank retrieval
- [ ] 2.1 Create `_search_images_graph_rerank(element_id, element_name, visual_keywords, embeddings, metadata, emb_model, top_k=3)` function
- [ ] 2.2 Inside: call `kg_retrieval.get_images_for_material(material_id=element_id)` for graph recall
- [ ] 2.3 Filter embedding matrix to only graph-returned image paths
- [ ] 2.4 Embed query ("name + keywords") and compute cosine similarity against filtered embeddings
- [ ] 2.5 Return top-k `ImageReference` objects sorted by similarity score
- [ ] 2.6 Handle zero-graph-results: return empty list

### 3. Update attach_images node with configurable retrieval mode
- [ ] 3.1 Add `image_retrieval_mode: str = "kg_rerank"` to the graph's `configurable` field (supports `"kg_rerank"` | `"embedding_only"`)
- [ ] 3.2 In `attach_images`, read `image_retrieval_mode` from `config["configurable"]` and branch accordingly
- [ ] 3.3 When `"kg_rerank"`: call `_search_images_graph_rerank()` with the Neo4j driver
- [ ] 3.4 When `"embedding_only"`: use existing full-index embedding search (preserve current logic)
- [ ] 3.5 Pass or initialize Neo4j driver within the node (only used in `"kg_rerank"` mode)

### 4. Update notebook & validate
- [ ] 4.1 Update `%%writefile` cell in `notebooks/7_material_recommender.ipynb`
- [ ] 4.2 Run `ruff check` on generated source
- [ ] 4.3 Test with `image_retrieval_mode="kg_rerank"`: verify images returned are KG-connected to the element
- [ ] 4.4 Test with `image_retrieval_mode="embedding_only"`: verify existing behavior preserved
- [ ] 4.5 Commit to development branch
