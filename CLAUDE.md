# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepTutor is an AI-powered personalized learning assistant built with Python (FastAPI backend) and Next.js (frontend). It features multi-agent collaboration for problem-solving, question generation, deep research, and interactive learning.

## Build & Run Commands

### Quick Start (Development)
```bash
# Start both backend and frontend
python scripts/start_web.py

# Start backend only (CLI mode)
python scripts/start.py

# Or using Docker
docker compose up --build -d
```

### Backend Only
```bash
# Run FastAPI server directly
python src/api/run_server.py

# Or with uvicorn
uvicorn src.api.main:app --host 0.0.0.0 --port 8001 --reload
```

### Frontend Only
```bash
cd web
npm install
npm run dev      # Development with Turbopack
npm run build    # Production build
npm run lint     # ESLint check
```

### Install Dependencies
```bash
# Python dependencies
pip install -r requirements.txt

# Frontend dependencies
npm install --prefix web

# Or use the install script
bash scripts/install_all.sh
```

### Pre-commit Hooks
```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

## Knowledge Base Management

```bash
# List all knowledge bases
python -m src.knowledge.start_kb list

# Create new knowledge base from documents
python -m src.knowledge.start_kb init my_kb --docs document.pdf

# Add documents incrementally (preferred over recreating)
python -m src.knowledge.add_documents my_kb --docs new_doc.pdf

# Extract numbered items (Definitions, Theorems, etc.)
python -m src.knowledge.start_kb extract --kb my_kb
```

## Architecture

### Directory Structure
```
src/
├── api/           # FastAPI backend with REST + WebSocket endpoints
│   ├── main.py    # FastAPI app setup, CORS, static files
│   └── routers/   # Endpoint modules (solve, question, research, etc.)
├── agents/        # Multi-agent system implementations
│   ├── solve/     # Problem-solving (dual-loop: Analysis + Solve)
│   ├── question/  # Question generation with validation
│   ├── research/  # Deep research pipeline (Planning → Researching → Reporting)
│   ├── guide/     # Guided learning with interactive pages
│   ├── co_writer/ # AI-assisted writing with RAG/web context
│   └── ideagen/   # Research idea generation
├── core/          # Configuration, logging, system initialization
│   ├── core.py    # Config loading: get_llm_config(), get_agent_params()
│   └── logging/   # Logger, LLM stats tracking
├── knowledge/     # Knowledge base management (LightRAG integration)
└── tools/         # Shared tools (RAG, web search, code execution)

web/               # Next.js 16 frontend (React 19, TailwindCSS)
config/
├── main.yaml      # System config (ports, paths, tools, module settings)
└── agents.yaml    # LLM parameters (temperature, max_tokens per module)
```

### Key Architectural Patterns

1. **Agent Parameters**: Always use `get_agent_params("module_name")` from `src/core/core.py` - never hardcode temperature/max_tokens.

2. **Configuration Hierarchy**: Environment variables (`.env`) → `agents.yaml` (LLM params) → `main.yaml` (all other settings).

3. **WebSocket Streaming**: Long-running operations (solve, research, question generation) use WebSocket endpoints for real-time progress.

4. **Knowledge Base**: Uses LightRAG for hybrid retrieval (vector + knowledge graph). Knowledge bases stored in `data/knowledge_bases/`.

## Configuration

### Environment Variables (Required in .env)
```bash
LLM_MODEL=gpt-4o                           # Model name
LLM_BINDING_API_KEY=your_key               # LLM API key
LLM_BINDING_HOST=https://api.openai.com/v1 # API endpoint
EMBEDDING_MODEL=text-embedding-3-large
EMBEDDING_BINDING_API_KEY=your_key
EMBEDDING_BINDING_HOST=https://api.openai.com/v1
```

### Config Files
- `config/main.yaml`: Ports (default: backend 8001, frontend 3782), paths, tool settings, module-specific non-LLM parameters
- `config/agents.yaml`: LLM temperature and max_tokens per module (solve, research, question, etc.)

## Code Style

- **Python**: Ruff for linting/formatting (line length 100), configured in `pyproject.toml`
- **Frontend**: Prettier for formatting
- **Security**: detect-secrets for scanning

### Commit Message Format
```
<type>: <short description>

Types: feat, fix, docs, style, refactor, test, chore
```

## API Endpoints

### WebSocket (Real-time streaming)
- `WS /api/v1/solve` - Problem solving
- `WS /api/v1/question/generate` - Question generation
- `WS /api/v1/research/run` - Deep research

### REST
- `GET /api/v1/knowledge/list` - List knowledge bases
- `POST /api/v1/knowledge/create` - Create knowledge base
- Static files: `/api/outputs/` serves `data/user/`

## Data Storage

All outputs are stored in `data/user/`:
- `solve/solve_YYYYMMDD_HHMMSS/` - Problem solutions
- `question/` - Generated questions
- `research/reports/` - Research reports
- `notebook/` - Notebook records
- `guide/` - Learning session state
