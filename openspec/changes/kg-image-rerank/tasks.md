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

### 3. Update attach_images node
- [ ] 3.1 Replace current embedding-only search with call to _search_images_graph_rerank
- [ ] 3.2 Pass Neo4j driver through state or initialize within node

### 4. Update notebook & validate
- [ ] 4.1 Update `%%writefile` cell in `notebooks/7_material_recommender.ipynb`
- [ ] 4.2 Run `ruff check` on generated source
- [ ] 4.3 Test with a sample recommendation: verify images returned are KG-connected to the element
- [ ] 4.4 Commit to development branch
