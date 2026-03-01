# HealthGuard AI

AI-powered parallel health data research platform. 5 specialized agents query CDC, FDA, MedlinePlus, and Healthcare.gov simultaneously and synthesize findings into a professional report.

## Architecture

```
User Query → FastAPI → 5 Parallel Agents → Synthesized Report → Streamed to Browser
```

| Agent | Data Source | Port |
|-------|------------|------|
| 🌍 EPHT_Agent | CDC Environmental Public Health Tracking | 8889 |
| 📊 OpenData_Agent | CDC Open Data (COVID, disease surveillance) | 8890 |
| 🏥 HealthcareAccess_Agent | Healthcare.gov (insurance, providers) | 8891 |
| 🩺 MedlinePlus_Agent | MedlinePlus Connect (patient education, ICD-10) | 8892 |
| ⚠️ OpenFDA_Agent | OpenFDA (FAERS adverse events, recalls, MAUDE) | 8893 |

## Quick Start

### Option A — Docker (recommended)

```bash
cp .env.example .env        # add your ANTHROPIC_API_KEY
docker-compose up --build
open frontend.html          # open in browser, connects to localhost:8000
```

### Option B — Local (uv)

```bash
cp .env.example .env        # add your ANTHROPIC_API_KEY
uv sync

# Terminal 1
uv run start_servers.py     # starts all 5 MCP servers (ports 8889–8893)

# Terminal 2
uv run fastapi_server.py    # API server on http://localhost:8000

# Open frontend.html in your browser
```

### Option C — Local (pip)

```bash
cp .env.example .env
pip install -r requirements.txt
python start_servers.py     # Terminal 1
python fastapi_server.py    # Terminal 2
open frontend.html
```

## Project Structure

```
├── fastapi_server.py           # Main API — agent orchestration + SSE streaming
├── start_servers.py            # Starts all 5 MCP servers
├── frontend.html               # React UI — 5 agent columns + final report
│
├── mcp_server_epht.py          # CDC EPHT MCP server
├── mcp_server_opendata.py      # CDC Open Data MCP server
├── mcp_server_healthcare.py    # Healthcare.gov MCP server
├── mcp_server_medlineplus.py   # MedlinePlus Connect MCP server
├── mcp_server_openfda.py       # OpenFDA MCP server
│
├── tools/                      # Data source API integrations
│   ├── cdc_epht.py
│   ├── cdc_open_data.py
│   ├── healthcare_gov.py
│   ├── medlineplus_connect.py
│   └── openfda_api.py
│
├── tests/                      # Integration tests
├── docs/                       # Supplementary documentation
│
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── pyproject.toml
└── requirements.txt
```

## API Endpoints

| Method | Endpoint | Description |
|--------|---------|-------------|
| `POST` | `/chat` | Stream research (SSE) — body: `{ "message": "..." }` |
| `GET` | `/health` | Health check |
| `GET` | `/agents` | List configured agents |

## Example Queries

- "What are the latest FDA recalls for blood pressure medications and what patient education is available?"
- "Analyze fentanyl-related adverse events and provide opioid safety education resources"
- "Compare air quality trends with respiratory health outcomes across states"
- "Which counties have the highest uninsured rates and how do COVID outcomes correlate?"

## Adding a New Data Source

1. Create `tools/your_source.py` with a `register_your_source_tools(mcp)` function
2. Create `mcp_server_your_source.py` — copy any existing one as a template
3. Add an `AgentConfig` entry to `AGENT_CONFIGS` in `fastapi_server.py`
4. Add the server to `start_servers.py` and `docker-compose.yml`
5. Add a column for it in `frontend.html`

## Tech Stack

- **Backend:** FastAPI, Server-Sent Events, asyncio
- **Agents:** Anthropic Claude claude-sonnet-4-6 (direct API)
- **Tools:** MCP protocol via `fastmcp`
- **Frontend:** React (CDN), no build step
- **Package manager:** uv
