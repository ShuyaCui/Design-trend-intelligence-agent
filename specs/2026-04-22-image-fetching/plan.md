# Implementation Plan: Research Agent Image Fetching

## Overview

Add image fetching capability to the research pipeline. Changes span state schemas, utility functions, agent nodes, prompts, and the final report writer. All code changes are made in notebooks and regenerated to `src/`.

---

## Group 1 — Schema & State Foundation

Lay the groundwork by defining the data model and threading it through all state objects.

### 1.1 Define `ImageResult` Pydantic schema
- **File**: `notebooks/2_research_agent.ipynb` → `state_research.py`
- Add `ImageResult(BaseModel)` with fields: `url`, `title`, `source_page`, `description`, `local_path` (optional)
- This schema is the single source of truth for image metadata

### 1.2 Add `images` field to `ResearcherState` and `ResearcherOutputState`
- **File**: `notebooks/2_research_agent.ipynb` → `state_research.py`
- `images: Annotated[list[ImageResult], operator.add]` with default `[]`
- Include in output state so images flow to supervisor

### 1.3 Add `images` field to `SupervisorState`
- **File**: `notebooks/4_research_supervisor.ipynb` → `state_multi_agent_supervisor.py`
- Same reducer pattern (`operator.add`)

### 1.4 Add `images` field to `AgentState`
- **File**: `notebooks/5_full_agent.ipynb` → `state_scope.py`
- Same reducer pattern; this is the top-level state for the full pipeline

---

## Group 2 — Image Collection in Search Tool

Modify the Tavily search tool to capture image URLs from search results.

### 2.1 Update `tavily_search_multiple()` to pass `include_images=True`
- **File**: `notebooks/2_research_agent.ipynb` → `utils.py`
- Add `include_images` parameter (default `True`)
- Return images alongside existing results

### 2.2 Update `tavily_search` tool to extract and return image metadata
- **File**: `notebooks/2_research_agent.ipynb` → `utils.py`
- Parse `images` from Tavily response
- Build `ImageResult` objects
- Include image info in formatted tool output so the LLM sees them

### 2.3 Add `download_images()` utility function
- **File**: `notebooks/2_research_agent.ipynb` → `utils.py`
- Download images from URLs to `reports/<session_id>/images/`
- Best-effort with timeout (5s per image)
- Respect `DISABLE_SSL_VERIFY` setting
- Return updated `ImageResult` objects with `local_path` populated
- Write `images_metadata.json` alongside downloaded files

---

## Group 3 — Agent Node Integration

Wire image data through the agent graph nodes.

### 3.1 Update `tool_node` in research agent to accumulate images
- **File**: `notebooks/2_research_agent.ipynb` → `research_agent.py`
- After tool execution, extract `ImageResult` objects from tool output
- Return `images` in state update

### 3.2 Update `compress_research` to preserve image references
- **File**: `notebooks/2_research_agent.ipynb` → `research_agent.py`
- Include image metadata in the context sent to the compression model
- Ensure compressed output retains image references

### 3.3 Update supervisor `supervisor_tools` to aggregate images from sub-agents
- **File**: `notebooks/4_research_supervisor.ipynb` → `multi_agent_supervisor.py`
- When `asyncio.gather()` returns sub-agent results, collect `images` from each
- Merge into supervisor state

---

## Group 4 — Prompt Updates

Update prompt templates to guide agents in image collection behavior.

### 4.1 Update `research_agent_prompt`
- **File**: `notebooks/2_research_agent.ipynb` → `prompts.py`
- Instruct the agent to note relevant images from search results
- Guide the agent on when images are useful (charts, diagrams, data viz)

### 4.2 Update `compress_research_system_prompt`
- **File**: `notebooks/2_research_agent.ipynb` → `prompts.py`
- Instruct compression to preserve image references with context

### 4.3 Update `final_report_generation_prompt`
- **File**: `notebooks/5_full_agent.ipynb` → `prompts.py`
- Instruct the writer to embed relevant images using Markdown syntax
- Provide image metadata as additional context

---

## Group 5 — Report Generation & Download

Integrate images into the final report output.

### 5.1 Update `final_report_generation` node
- **File**: `notebooks/5_full_agent.ipynb` → `research_agent_full.py`
- Pass image metadata to the report generation prompt
- Trigger image download to `reports/<session_id>/images/`

### 5.2 Write `images_metadata.json` alongside report
- **File**: `notebooks/5_full_agent.ipynb` → `research_agent_full.py`
- Persist structured image metadata for downstream consumers (e.g., Web UI)

---

## Group 6 — Validation & Cleanup

### 6.1 Run `ruff check src/` and fix any lint issues in notebooks
### 6.2 Verify backward compatibility: run research without images scenario
### 6.3 Update `specs/roadmap.md`, `specs/tech-stack.md`, `specs/mission.md`
### 6.4 Commit to `development` branch
