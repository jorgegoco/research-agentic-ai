# 📚 Reflective Research Agent

> An AI-powered multi-agent system that autonomously conducts research, writes comprehensive academic reports, and provides real-time progress tracking.

A FastAPI web application that orchestrates intelligent agents to perform automated research. The system uses a planning agent to decompose research tasks, executes specialized agents (research, writer, editor) with tool-calling capabilities, and stores all task state and results in PostgreSQL—all running in a single Docker container for simplified deployment.

---

## 📑 Table of Contents

- [Features](#-features)
- [Architecture](#-architecture)
- [Quick Start](#-quick-start)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Usage Guide](#-usage-guide)
- [Development](#-development)
- [Customization](#-customization)
- [Troubleshooting](#-troubleshooting)
- [Project Structure](#-project-structure)

---

## ✨ Features

### Core Capabilities

- **🤖 Multi-Agent System**: Orchestrates planner, research, writer, and editor agents working in sequence
- **🔍 Advanced Research Tools**:
  - Tavily web search for current information
  - arXiv integration with PDF extraction for academic papers
  - Wikipedia for background and definitions
- **📝 Academic Report Generation**: Produces publication-ready reports with proper citations and references
- **⚡ Real-time Progress Tracking**: Live updates on each step of the research workflow
- **💾 Persistent Storage**: PostgreSQL database stores all tasks, results, and execution history
- **🌐 Web Interface**: User-friendly UI for submitting queries and monitoring progress
- **🔌 RESTful API**: Full API access for programmatic integration

### Agent Capabilities

- **Planning Agent** (OpenAI o4-mini): Creates structured 6-7 step research plans
- **Research Agent** (GPT-4.1-mini): Intelligently selects and uses multiple research tools
- **Writer Agent** (GPT-4.1-mini): Drafts comprehensive academic reports with proper structure
- **Editor Agent** (GPT-4.1-mini): Reviews, refines, and improves draft quality

---

## 🏗️ Architecture

### Multi-Agent Workflow

```mermaid
graph TD
    A[User Submits Research Prompt] --> B[Planning Agent<br/>OpenAI o4-mini]
    B --> C{Generate Execution Plan<br/>Max 7 Steps}

    C --> D[Step 1: Tavily Search]
    C --> E[Step 2: arXiv Search]
    C --> F[Step 3-5: Additional Research]
    C --> G[Step 6: Draft Report]
    C --> H[Step 7: Edit Report]

    D --> I[Executor Agent]
    E --> I
    F --> I
    G --> I
    H --> I

    I --> J{Route to<br/>Specialized Agent}

    J -->|Research Task| K[Research Agent<br/>GPT-4.1-mini]
    J -->|Write Task| L[Writer Agent<br/>GPT-4.1-mini]
    J -->|Edit Task| M[Editor Agent<br/>GPT-4.1-mini]

    K --> N[Tool Selection]
    N --> O[Tavily<br/>Web Search]
    N --> P[arXiv<br/>Academic Papers + PDF]
    N --> Q[Wikipedia<br/>Background Info]

    O --> R[Collect Results]
    P --> R
    Q --> R

    R --> S[Build Execution History]

    L --> S
    M --> S

    S --> T{More Steps?}
    T -->|Yes| I
    T -->|No| U[Final Academic Report<br/>with Citations]

    U --> V[Store in PostgreSQL]
    V --> W[Display in Web UI]

    style A fill:#e1f5ff,stroke:#0066cc,stroke-width:3px,color:#000
    style B fill:#fff4e1,stroke:#cc8800,stroke-width:3px,color:#000
    style K fill:#ffe1f5,stroke:#cc0066,stroke-width:3px,color:#000
    style L fill:#ffe1f5,stroke:#cc0066,stroke-width:3px,color:#000
    style M fill:#ffe1f5,stroke:#cc0066,stroke-width:3px,color:#000
    style U fill:#e1ffe1,stroke:#00aa00,stroke-width:3px,color:#000
    style V fill:#f0f0f0,stroke:#666666,stroke-width:3px,color:#000
    style W fill:#e1f5ff,stroke:#0066cc,stroke-width:3px,color:#000
```

### Technology Stack

- **Backend**: FastAPI + Uvicorn
- **Database**: PostgreSQL 15/17
- **AI Models**: OpenAI GPT-4.1-mini, o4-mini (via aisuite)
- **Research APIs**: Tavily, arXiv, Wikipedia
- **PDF Processing**: PyMuPDF, pdfminer.six
- **Frontend**: Jinja2 templates, Bootstrap 5, marked.js
- **Deployment**: Docker (single-container setup)

---

## 🚀 Quick Start

### For First-Time Users

**1. Clone and Navigate:**

```bash
git clone <your-repo-url>
cd research-agentic-ai
```

**2. Create `.env` file:**

```bash
cat > .env << EOF
OPENAI_API_KEY=sk-your-openai-key-here
TAVILY_API_KEY=tvly-your-tavily-key-here
DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
EOF
```

**3. Build Docker Image:**

```bash
docker build -t fastapi-postgres-service .
```

**4. Run the Application:**

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

**5. Open in Browser:**

- Navigate to: http://localhost:8000
- Submit a research query and watch the magic happen! ✨

---

## 📋 Prerequisites

### Required

- **Docker**: Docker Desktop (Windows/macOS) or Docker Engine (Linux/WSL2)
  - Installation: https://docs.docker.com/get-docker/
  - For WSL2 Ubuntu: `sudo apt-get install docker.io docker-compose`

### API Keys

You'll need accounts and API keys from:

1. **OpenAI** (https://platform.openai.com/api-keys)

   - Used for AI model access (GPT-4.1-mini, o4-mini)
   - Pricing: ~$0.15 per 1M input tokens

2. **Tavily** (https://tavily.com/)
   - Used for web search capabilities
   - Free tier available

### System Requirements

- **RAM**: 4GB minimum, 8GB recommended
- **Disk Space**: ~2GB for Docker images
- **Ports**: 8000 (FastAPI), 5432 (PostgreSQL) must be available

---

## 💻 Installation

### Docker Setup (Recommended)

**For WSL2 Ubuntu:**

```bash
# Install Docker
sudo apt-get update
sudo apt-get install -y docker.io docker-compose

# Add your user to docker group
sudo usermod -aG docker $USER

# Start Docker service
sudo service docker start

# Refresh group membership
newgrp docker

# Verify installation
docker --version
docker ps
```

**Build the Image:**

```bash
cd /path/to/research-agentic-ai
docker build -t fastapi-postgres-service .
```

This process:

- Downloads Python 3.11 base image
- Installs PostgreSQL and system dependencies
- Installs Python packages from requirements.txt
- Copies your application code
- Takes 3-5 minutes on first build

**Verify Build:**

```bash
docker images
# Should show: fastapi-postgres-service  latest  ...
```

---

## 📖 Usage Guide

### A. Using the Web Interface

#### 1. Submit Research Tasks

- **URL**: http://localhost:8000
- **Steps**:
  1. Enter a detailed research prompt (3-4 sentences works best)
  2. Click "Submit"
  3. Watch the steps execute in real-time
  4. Expand steps with ➕ to see detailed agent activity
  5. Download the final report as .md or .html

#### 2. Effective Prompt Examples

**Example 1 - Technical Topic:**

```
Large language models for code generation. Focus on approaches like Codex,
AlphaCode, and CodeLlama. Include recent benchmarks on HumanEval and MBPP
datasets, and discuss fine-tuning techniques for domain-specific coding tasks.
```

**Example 2 - Scientific Discovery:**

```
Applications of deep learning in drug discovery, particularly in molecular
property prediction and de novo drug design. Cover graph neural networks,
transformers for SMILES representations, and recent successful case studies
from 2023-2024.
```

**Example 3 - Theoretical CS:**

```
Recent advances in quantum computing algorithms for optimization problems.
Include Grover's algorithm, quantum annealing, and QAOA. Compare performance
against classical algorithms and discuss current hardware limitations.
```

**Example 4 - Interdisciplinary:**

```
The role of AI in climate change modeling and prediction. Focus on machine
learning applications in weather forecasting, climate pattern recognition,
and carbon emissions optimization. Include recent breakthroughs and limitations.
```

#### 3. Monitor Progress

The interface provides real-time updates on:

- **Step Status**: pending → running → done/error
- **Agent Activity**: Which agent is executing (Research/Writer/Editor)
- **Tool Usage**: See which tools were called (Tavily, arXiv, Wikipedia)
- **Substep Details**: Click ➕ to expand and see:
  - User prompt context
  - Previous step outputs
  - Current task description
  - Agent output with tool calls

#### 4. Download Reports

Once complete:

- **Markdown Format**: Click "⬇️ Download .md"

  - Plain text format with markdown syntax
  - Great for version control, editing, or conversion

- **HTML Format**: Click "⬇️ Download .html"
  - Rendered HTML with styling
  - Ready for web publishing

Reports include:

- Full academic structure (Abstract, Introduction, Methods, Results, Discussion, Conclusion)
- Inline citations [1], [2], etc.
- Complete References section with clickable links
- Proper formatting and professional tone

---

### B. Using the API Directly

For programmatic access or automation, use the REST API.

#### 1. Submit a Research Task

```bash
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Quantum computing applications in cryptography"}' \
  | jq
```

**Response:**

```json
{
  "task_id": "abc123-def456-ghi789"
}
```

#### 2. Check Progress

```bash
curl http://localhost:8000/task_progress/abc123-def456-ghi789 | jq
```

**Response Structure:**

```json
{
  "steps": [
    {
      "title": "Research agent: Use Tavily...",
      "status": "done",
      "description": "Completed: Research agent...",
      "substeps": [...]
    }
  ]
}
```

#### 3. Get Final Report

```bash
curl http://localhost:8000/task_status/abc123-def456-ghi789 | jq
```

**Response:**

```json
{
  "status": "done",
  "result": {
    "html_report": "# Research Report Title\n\n## Abstract\n...",
    "history": [...]
  }
}
```

#### 4. Health Check

```bash
curl http://localhost:8000/api
```

**Response:**

```json
{
  "status": "ok"
}
```

#### 5. Interactive API Docs

FastAPI provides automatic interactive documentation:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

#### 6. Understanding the Request Flow

Here's how the system handles research requests from start to finish:

```mermaid
sequenceDiagram
    participant User
    participant FastAPI
    participant DB as PostgreSQL
    participant Memory as In-Memory Dict
    participant BG as Background Thread
    participant Planner as Planning Agent
    participant Executor
    participant Agent as Specialized Agent
    participant Tools as Research Tools

    Note over User,Tools: 1. Initial Request
    User->>FastAPI: POST /generate_report<br/>{prompt: "research topic"}
    FastAPI->>DB: Create Task<br/>(id, status: "running")
    FastAPI->>Memory: Initialize task_progress[id]
    FastAPI->>BG: Start workflow thread
    FastAPI-->>User: Return {task_id: "abc123..."}

    Note over User,Tools: 2. Progress Polling
    User->>FastAPI: GET /task_progress/abc123
    FastAPI->>Memory: Fetch task_progress[abc123]
    FastAPI-->>User: Return {steps: [...]}

    Note over BG,Tools: 3. Background Execution
    BG->>Planner: Generate execution plan
    Planner-->>BG: Return [Step1...Step7]

    loop For each step (1-7)
        BG->>Memory: Update step status: "running"
        BG->>Executor: Execute step with history
        Executor->>Agent: Route to Research/Writer/Editor

        alt Research Task
            Agent->>Tools: Call Tavily/arXiv/Wikipedia
            Tools-->>Agent: Return search results
        else Writing Task
            Agent->>Agent: Generate report from research
        else Editing Task
            Agent->>Agent: Review and improve draft
        end

        Agent-->>Executor: Return output
        Executor->>Memory: Update step status: "done"
        Executor->>BG: Add to execution history
    end

    BG->>DB: Update Task<br/>(status: "done", result: JSON)

    Note over User,Tools: 4. Final Retrieval
    User->>FastAPI: GET /task_status/abc123
    FastAPI->>DB: Query Task by ID
    DB-->>FastAPI: Return Task record
    FastAPI-->>User: Return {status: "done",<br/>result: {html_report, history}}

    Note over User,FastAPI: User downloads .md or .html report
```

**Key Points:**

- **Asynchronous Design**: The API returns immediately with a `task_id`, while research runs in the background
- **Two Storage Layers**:
  - PostgreSQL for persistent task storage
  - In-memory dict for real-time progress updates
- **Polling Pattern**: Frontend polls `/task_progress` every 2 seconds for live updates
- **Execution History**: Each step's output is passed to subsequent steps for context

---

### C. Database Access

Direct access to PostgreSQL for advanced users.

> **Note:** This application is designed to run in a Docker container with PostgreSQL bundled inside. You don't need PostgreSQL installed on your host system. The database is accessed through the running container.

#### Connect to Database

**Recommended: Access from inside the Docker container (no installation needed)**

```bash
# Make sure the container is running
docker ps

# Connect to the database from inside the container
docker exec -it fpsvc psql "postgresql://app:local@127.0.0.1:5432/appdb"
```

**Alternative: Install PostgreSQL client on host (optional)**

If you prefer to connect from your WSL2/host terminal directly:

```bash
# Install PostgreSQL client tools only (not the full server)
sudo apt update && sudo apt install -y postgresql-client

# Connect from host (container must be running with port 5432 exposed)
psql "postgresql://app:local@localhost:5432/appdb"
```

#### Useful Queries

```sql
-- List all tasks
SELECT id, status, created_at FROM tasks ORDER BY created_at DESC;

-- Get a specific task
SELECT * FROM tasks WHERE id = 'abc123-def456-ghi789';

-- Count tasks by status
SELECT status, COUNT(*) FROM tasks GROUP BY status;

-- Find recent completed tasks
SELECT id, prompt, created_at
FROM tasks
WHERE status = 'done'
ORDER BY created_at DESC
LIMIT 10;

-- Exit psql
\q
```

#### Database Schema

Visual representation of the data model:

```mermaid
erDiagram
    TASKS {
        varchar id PK "UUID, Primary Key"
        text prompt "User's research query"
        varchar status "running | done | error"
        timestamp created_at "Task creation time"
        timestamp updated_at "Last modification time"
        text result "JSON: {html_report, history}"
    }

    TASK_PROGRESS {
        varchar task_id FK "References tasks.id"
        json steps "Array of step objects"
        string title "Step description"
        string status "pending | running | done | error"
        string description "Current activity"
        json substeps "Detailed execution info"
        timestamp updated_at "Last update time"
    }

    TASKS ||--o{ TASK_PROGRESS : "tracks"
```

**Storage Strategy:**

| Component              | Storage Type            | Purpose                                               | Lifetime                            |
| ---------------------- | ----------------------- | ----------------------------------------------------- | ----------------------------------- |
| **Tasks Table**        | PostgreSQL              | Persistent storage of task metadata and final reports | Permanent (unless manually deleted) |
| **task_progress Dict** | In-Memory (Python dict) | Real-time progress tracking for active tasks          | Process lifetime (lost on restart)  |

**Key Schema Details:**

1. **Tasks Table** (`main.py:39-46`):

   - **id**: UUID generated by `uuid.uuid4()`
   - **prompt**: Original user query (TEXT, no length limit)
   - **status**: Current state (`running` → `done` or `error`)
   - **result**: JSON string containing:
     ```json
     {
       "html_report": "Markdown text...",
       "history": [{"title": "...", "status": "...", "substeps": [...]}]
     }
     ```

2. **task_progress In-Memory Dict** (`main.py:67, 92-102`):
   - Key: `task_id` (string)
   - Value: `{"steps": [...]}`
   - Updated in real-time during execution
   - Enables live progress tracking without DB overhead

**Important Notes:**

- ⚠️ **Database drops all tables on startup** by default (`main.py:50`)
- To persist data between restarts, see [Customization → Persistent Database Storage](#1-persistent-database-storage)
- In-memory progress is lost on container restart (fetch from `tasks.result` instead)

---

### D. Managing the Application

#### Start the Application

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service
```

**Expected Output:**

```
🚀 Starting Postgres cluster 17/main...
✅ Postgres is ready
CREATE ROLE
CREATE DATABASE
🔗 DATABASE_URL=postgresql://app:local@127.0.0.1:5432/appdb
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

#### Stop the Application

Press `Ctrl + C` for graceful shutdown

#### Check Running Containers

```bash
docker ps
```

#### View Logs

```bash
docker logs -f fpsvc
```

---

## 🛠️ Development

### Hot Reload Mode

For active development, mount your code directory and enable auto-reload:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -p 5432:5432 \
  -v "$PWD":/app \
  --name fpsvc \
  --env-file .env \
  fastapi-postgres-service \
  bash -lc "pg_ctlcluster 17 main start && uvicorn main:app --host 0.0.0.0 --port 8000 --reload"
```

**Benefits:**

- Edit Python files and see changes immediately
- No need to rebuild Docker image
- Faster iteration during development

### Rebuild After Changes

**When to rebuild:**

- Modified `Dockerfile`
- Updated `requirements.txt`
- Changed `docker/entrypoint.sh`

**How to rebuild:**

```bash
docker build -t fastapi-postgres-service .
```

---

## ⚙️ Customization

### 1. Persistent Database Storage

**Problem**: By default, database drops all tables on startup (main.py:50)

**Solution**: Comment out the drop statement in `main.py`:

```python
# Base.metadata.drop_all(bind=engine)  # Comment this line
Base.metadata.create_all(bind=engine)
```

Then rebuild:

```bash
docker build -t fastapi-postgres-service .
```

### 2. Adjust PDF Extraction Settings

Edit `src/research_tools.py` (lines 160-166):

```python
_INCLUDE_PDF = True        # Set False to skip PDF download
_EXTRACT_TEXT = True       # Set False to skip text extraction
_MAX_PAGES = 10            # Increase to extract more pages (default: 6)
_TEXT_CHARS = 10000        # Increase for more text (default: 5000)
```

**Trade-offs:**

- More pages/chars = Better context but slower execution
- Fewer pages/chars = Faster but may miss important details

### 3. Change AI Models

The system uses `aisuite` for unified model access.

**Planning Agent** (src/planning_agent.py:27):

```python
def planner_agent(topic: str, model: str = "openai:o4-mini"):
```

**Research/Writer/Editor Agents** (src/agents.py):

```python
def research_agent(prompt: str, model: str = "openai:gpt-4.1-mini", ...):
def writer_agent(prompt: str, model: str = "openai:gpt-4.1-mini", ...):
def editor_agent(prompt: str, model: str = "openai:gpt-4.1-mini", ...):
```

**Available Models:**

```python
# OpenAI
"openai:gpt-4o"           # Most capable
"openai:gpt-4o-mini"      # Balanced
"openai:gpt-4.1-mini"     # Current default

# Anthropic (requires ANTHROPIC_API_KEY)
"anthropic:claude-3-5-sonnet-20241022"

# Google (requires GOOGLE_API_KEY)
"google:gemini-1.5-pro"
```

### 4. Add New Research Tools

**Example: Add Google Scholar**

1. Create tool function in `src/research_tools.py`
2. Add to tools list in `src/agents.py:80`
3. Update system prompt
4. Rebuild and test

---

## 🔧 Troubleshooting

### Port Already in Use

**Error**: `address already in use`

**Solution:**

```bash
sudo lsof -i :8000
sudo kill -9 <PID>
```

### API Keys Not Working

**Symptom**: `TAVILY_API_KEY not found`

**Solution:**

1. Verify `.env` file exists
2. Check keys have no quotes
3. Restart container with `--env-file .env`

### Out of Memory

**Solutions:**

- Reduce PDF extraction pages/chars in `src/research_tools.py`
- Use shorter prompts
- Increase Docker memory allocation

### Tables Disappear on Restart

**Cause**: `main.py:50` drops tables on startup

**Solution**: See [Customization → Persistent Database Storage](#1-persistent-database-storage)

---

## 📁 Project Structure

```
research-agentic-ai/
├── main.py                      # FastAPI app, API endpoints, DB setup
├── src/
│   ├── planning_agent.py        # Planning and execution orchestration
│   ├── agents.py                # Research, Writer, Editor agents
│   └── research_tools.py        # Tavily, arXiv, Wikipedia tools
├── templates/
│   └── index.html               # Web UI with progress tracking
├── static/                      # Static assets
├── docker/
│   └── entrypoint.sh            # Container startup script
├── requirements.txt             # Python dependencies
├── Dockerfile                   # Docker image definition
├── .env                         # API keys (not in git)
├── .gitignore
└── README.md                    # This file
```

### Key Files

**`main.py`** (232 lines)

- FastAPI initialization
- Database model (Task table)
- API endpoints
- Background workflow execution

**`src/planning_agent.py`** (177 lines)

- OpenAI o4-mini planning
- Workflow enforcement
- Maximum 7 steps

**`src/agents.py`** (297 lines)

- Research agent with tool calling
- Writer agent (1500-3000 word reports)
- Editor agent (quality improvement)

**`src/research_tools.py`** (408 lines)

- arXiv: PDF download + text extraction
- Tavily: Web search
- Wikipedia: Article summaries

---

## 📊 Quick Reference

```bash
# Build
docker build -t fastapi-postgres-service .

# Run
docker run --rm -it -p 8000:8000 -p 5432:5432 --name fpsvc --env-file .env fastapi-postgres-service

# Access
http://localhost:8000

# Stop
Ctrl + C

# View logs
docker logs -f fpsvc

# Database
psql "postgresql://app:local@localhost:5432/appdb"

# Submit via API
curl -X POST http://localhost:8000/generate_report \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Your research topic"}'
```

---

**Happy Researching! 🧠✨**

_Built with FastAPI, PostgreSQL, OpenAI, and Docker_
