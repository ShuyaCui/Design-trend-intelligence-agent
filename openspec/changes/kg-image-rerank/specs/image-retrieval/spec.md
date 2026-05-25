## MODIFIED Requirements

### Requirement: Element-to-image retrieval strategy
The system SHALL change the retrieval strategy used by `attach_images` from pure embedding search over the full image index to a graph-first plus embedding-rerank flow. The function signature and output format SHALL remain unchanged.

#### Scenario: Retrieval executes for a recommended element
- **WHEN** `attach_images` looks up reference images for a recommended element
- **THEN** the system SHALL first recall graph-linked images for that element's `material_id`, rerank only those candidates with embedding cosine similarity, and return the result in the same `ImageReference` list format as before

#### Scenario: Callers depend on the existing interface
- **WHEN** existing recommender code invokes the image retrieval helper or consumes its results
- **THEN** no caller-visible signature change or output schema change SHALL be required
