## Context
The existing image attachment flow in `material_recommender.py` uses embedding-only retrieval inside the `attach_images` node. For each recommended element, the node builds a query from the element name and visual keywords, embeds it, and searches the full cached image embedding index for the closest matches. This can surface visually similar but semantically unrelated images.

The repository already has the pieces needed for a higher-precision hybrid approach: `kg_retrieval.get_images_for_material()` can recall images linked to a material in Neo4j, `material_library/image_embeddings.npz` already caches image embeddings and metadata, and `AzureOpenAIEmbeddings` is already initialized in `material_recommender.py`. The recommender graph itself already follows the established `StateGraph` + `TypedDict` architecture and is generated from `notebooks/7_material_recommender.ipynb` via a `%%writefile` cell.

## Goals / Non-Goals
### Goals
- Make KG recall + embedding reranking the **default** retrieval mode in `attach_images`, while preserving embedding-only retrieval as a selectable fallback via config.
- Reuse existing Neo4j retrieval, cached embeddings, and embedding model initialization.
- Preserve the current recommender graph shape and output contract.
- Keep the implementation aligned with the notebook-driven source generation workflow.

### Non-Goals
- Do not change recommendation generation in the `recommend` node.
- Do not change the KG schema, edge creation logic, or image embedding cache format.
- Do not add new dependencies or introduce a new retrieval subsystem.

## Decisions
- **Integration point**: Modify the retrieval helper used by `attach_images` in `material_recommender.py`; this is currently `_search_images_for_element()` or an equivalent helper in the generated module.
- **Retrieval mode config**: Introduce an `image_retrieval_mode` config parameter (e.g., `"kg_rerank"` | `"embedding_only"`). Default is `"kg_rerank"`. The `attach_images` node reads this from the graph's `configurable` field (standard LangGraph pattern).
- **KG recall + rerank flow**: For each recommended element, call `get_images_for_material(element_id)` to recall candidate image paths from the KG, filter the cached embedding matrix to those paths, compute cosine similarity between the query embedding and the filtered image embeddings, and return the top-N matches.
- **Embedding-only fallback flow**: When `image_retrieval_mode = "embedding_only"`, use the existing search over the full 275-image embedding index (current behavior). This preserves backward compatibility and allows comparison.
- **Query construction**: Keep the current query format: `element_name + visual_keywords`. This preserves compatibility with the existing embedding intent while improving recall precision through graph filtering.
- **Reuse of existing assets**: Reuse `kg_retrieval.get_images_for_material()`, the cached `image_embeddings.npz` index, and the already initialized `AzureOpenAIEmbeddings` client in `material_recommender.py`.
- **Zero-result behavior (KG mode)**: If the KG returns no images for a material, return an empty list. This trusts the graph as the source of truth because Image nodes already have 100% coverage.
- **Notebook workflow**: Implement the change in the `%%writefile` cell in `notebooks/7_material_recommender.ipynb`, then regenerate `material_recommender.py` from the notebook. Do not hand-edit generated `src/` files.
- **Dependencies and architecture**: Introduce no new dependencies and stay within the existing `StateGraph`, `TypedDict`, and notebook-to-source generation patterns.

## Risks / Trade-offs
- Trusting KG recall improves precision but removes the broader recall of the previous embedding-only search; if graph edges are missing or stale, image results will be empty. The `embedding_only` mode exists as an escape hatch.
- Filtering the embedding matrix by graph-returned paths assumes the KG image paths and cached metadata paths stay normalized and consistent.
- Passing or initializing the Neo4j driver for `attach_images` slightly increases coupling between the recommender graph and KG access.
- Keeping the current query format minimizes change risk, but the reranking quality still depends on the quality of `visual_keywords` and image descriptions.
- The `configurable` LangGraph pattern adds a small amount of boilerplate but is already established in the codebase.
