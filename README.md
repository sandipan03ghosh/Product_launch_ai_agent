# AI Product Launch Intelligence Agent

A multi-step AI agent system that generates product launch briefs using a graph-based workflow.

This project combines:
- Planning and orchestration with LangGraph
- LLM generation with Groq (LangChain integration)
- Optional web research with Tavily
- Optional image generation with Hugging Face Inference API
- A Streamlit frontend for interactive brief generation and past brief browsing
- LangSmith tracing support for observability

## What This Project Does

Given a launch context (for example: launching a new phone in India), the app:
1. Routes the request into a mode (`closed_book`, `hybrid`, or `open_book`)
2. Optionally runs web research and normalizes evidence
3. Builds a detailed strategy plan with multiple tasks
4. Fans out tasks to worker nodes that generate section markdown
5. Reduces/merges sections into a final brief
6. Optionally plans and generates supporting images
7. Saves the output as markdown and lets users download markdown/zip bundles

## Tech Stack

- Python
- Streamlit
- LangGraph
- LangChain Core
- Pydantic
- Groq via `langchain-groq`
- Tavily via `langchain-tavily`
- Hugging Face Inference API via `huggingface_hub`
- Pillow
- Pandas
- python-dotenv
- LangSmith (tracing)

## Project Structure

- `backend.py`: Core agent graph, schemas, routing, research, orchestration, worker generation, reducer/image pipeline
- `frontend.py`: Streamlit UI, run flow, progress display, preview/downloads, past brief loading
- `requirement.txt`: Python dependencies
- `images/`: Generated images used in briefs
- `*.md`: Generated launch brief outputs

## Agent Architecture (High Level)

Main graph flow:

`START -> router -> (research or orchestrator) -> fanout workers -> reducer subgraph -> END`

### Nodes

- `router_node`
  - Decides whether research is needed
  - Selects mode: `closed_book`, `hybrid`, or `open_book`
  - Sets recency constraints

- `research_node`
  - Runs Tavily queries when required
  - Extracts and deduplicates evidence
  - Applies recency filtering (especially in `open_book` mode)

- `orchestrator_node`
  - Produces structured plan (`Plan`) with 6-10 tasks
  - Defines audience, tone, blog kind, and constraints

- `worker_node`
  - Generates one markdown section per task
  - Applies grounding/citation constraints based on mode

- `reducer` subgraph
  1. `merge_content`
  2. `decide_images`
  3. `generate_and_place_images`

## Frontend UX

The Streamlit app includes:
- Bottom chat-style input for launch context submission
- Sidebar for past brief selection/loading
- Tabs for:
  - Strategy Plan
  - Market Evidence
  - Brief Preview
  - Visuals
  - Logs
- Download options:
  - Markdown only
  - Markdown + images bundle zip
  - Images zip

## Setup

### 1) Create and activate virtual environment (Windows PowerShell)

```powershell
cd D:\prod_launch_agent\product_agent
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2) Install dependencies

```powershell
pip install -r requirement.txt
```

### 3) Create `.env`

Create a `.env` file in project root and set required keys.

Minimal example:

```env
# LLM
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=llama-3.3-70b-versatile

# Research (optional but recommended for hybrid/open_book)
TAVILY_API_KEY=your_tavily_api_key

# Images (optional; needed for image generation)
HF_TOKEN=your_huggingface_token
# Optional override:
# HF_IMAGE_MODEL=stabilityai/stable-diffusion-xl-base-1.0

# LangSmith tracing (optional but recommended)
LANGSMITH_API_KEY=your_langsmith_key
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=product-launch-agent
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
```

Compatibility fallback (only if traces do not appear):

```env
LANGCHAIN_API_KEY=your_langsmith_key
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=product-launch-agent
LANGCHAIN_ENDPOINT=https://api.smith.langchain.com
```

## Run

```powershell
streamlit run frontend.py
```

Then open the local URL shown by Streamlit (typically `http://localhost:8501`).

## How To Use

1. Enter launch context in the bottom input bar and send.
2. Wait for graph execution and progress updates.
3. Review output in tabs:
   - Plan quality
   - Evidence coverage
   - Final brief markdown with local/remote image rendering
4. Download outputs as needed.
5. Use sidebar to load past generated briefs.

## Output Files

- Brief markdown file: `<slugified_title>.md`
- Generated images: `images/<filename>.png`
- Optional downloads via UI:
  - Markdown
  - Bundle zip (markdown + images)
  - Images zip

## LangSmith: What You Can Monitor

Once tracing is enabled:
- End-to-end runs
- Node-level execution paths
- Inputs/outputs per step
- Latency and token/cost insights (when available)
- Regression comparisons after prompt/code changes

## License

Add a project license before public release (for example: MIT).
