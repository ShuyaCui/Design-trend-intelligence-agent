## ADDED Requirements

### Requirement: MaterialElement JSON schema definition
The system SHALL define a canonical JSON schema for individual design elements with the following required fields: id (string), dimension (enum), name (string), name_en (string), visual_keywords (array of strings), aesthetic_persona (string), signals (array of strings), maturity (enum), year_range (string), typical_use (string), source_section (string).

#### Scenario: Valid element passes schema validation
- **WHEN** a JSON object contains all required fields with correct types
- **THEN** it is accepted as a valid MaterialElement

#### Scenario: Missing required field rejected
- **WHEN** a JSON object is missing the `aesthetic_persona` field
- **THEN** the system raises a validation error

### Requirement: Three-dimension classification
Every element MUST be classified into exactly one of three dimensions: "颜色", "装饰物", "透明度与质地".

#### Scenario: Dimension values are constrained
- **WHEN** an element is created with `dimension: "纹理"`
- **THEN** validation fails because "纹理" is not in the allowed enum

#### Scenario: Texture elements use combined dimension
- **WHEN** extracting an element about "凝胶感/啫喱感" or "高折光通透"
- **THEN** it is classified under "透明度与质地" (not a separate "质地" dimension)

### Requirement: Maturity level enum
Maturity MUST be one of: "主流", "上升", "实验性".

#### Scenario: Maturity values are constrained
- **WHEN** an element is created with `maturity: "emerging"`
- **THEN** validation fails because only Chinese enum values are accepted

### Requirement: Per-dimension output format
Output SHALL be organized by dimension, not by report. Each dimension file (`color.json`, `decoration.json`, `texture.json`) SHALL contain elements from ALL processed reports, grouped by maturity level ("主流", "上升", "实验性"), with each element retaining its `source_report` and `product_category` for traceability.

#### Scenario: Color dimension file structure
- **WHEN** `material_library/color.json` is generated from 3 reports
- **THEN** it contains a `主流` array, an `上升` array, and an `实验性` array, each containing color elements from all 3 reports

#### Scenario: Element traceability across reports
- **WHEN** an element "琥珀金/蜂蜜色" appears in both shampoo and serum reports
- **THEN** each instance appears in `color.json` with its respective `source_report` and `product_category`

### Requirement: Index metadata format
`index.json` SHALL track: list of processed reports (filename, extraction_date, element_count), per-dimension element counts, total element count, and last_updated timestamp.

#### Scenario: Index tracks all processed reports
- **WHEN** 3 reports are processed
- **THEN** `index.json` contains 3 entries in `processed_reports` with correct filenames and element counts

#### Scenario: Index is updated incrementally
- **WHEN** a 4th report is processed
- **THEN** `index.json` adds a 4th entry; dimension files are rebuilt to include elements from all 4 reports

### Requirement: Persona catalog with combinability
`personas.json` SHALL contain: a catalog of all aesthetic personas with descriptions, and for each persona, example element combinations spanning all 3 dimensions. This serves as the cross-dimensional combinability reference.

#### Scenario: Persona includes cross-dimension combinations
- **WHEN** `personas.json` is read for persona "科技净澈"
- **THEN** it lists typical color elements (e.g. "无色透明"), decoration elements (e.g. "微囊悬浮"), and texture elements (e.g. "高折光水感") that form a coherent combination

#### Scenario: All element personas reference the catalog
- **WHEN** any element's `aesthetic_persona` in `color.json`, `decoration.json`, or `texture.json` is checked
- **THEN** it matches a persona defined in `personas.json`

### Requirement: Aesthetic persona catalog
The system SHALL maintain a predefined persona catalog with at least 6 personas: 科技净澈, 天然奢养, 奢华克制, 感官甜品, 自然清体, 可视科技. Each persona SHALL have a description and typical element combinations.

#### Scenario: Persona catalog is included in output
- **WHEN** `cross_reference.json` is generated
- **THEN** it includes a `personas` section listing all personas with descriptions and example element combinations

#### Scenario: All elements reference valid personas
- **WHEN** any element's `aesthetic_persona` is checked
- **THEN** it matches one of the personas in the catalog (predefined or dynamically added)
