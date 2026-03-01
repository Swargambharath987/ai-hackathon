# HealthGuard AI — Project Context

## Overview
AI-powered parallel health data research platform for public health officials.
5 specialized agents query CDC, FDA, MedlinePlus, and Healthcare.gov simultaneously, then synthesize results into a comprehensive report.

## Repo
- **GitHub:** https://github.com/Swargambharath987/ai-hackathon
- **Branch:** `main` (only branch)
- **Local:** `~/ai-hackathon`
- **Original upstream:** `JZolton/ai-hackathon` (forked)

## Architecture

```
User Query → FastAPI (/chat SSE) → 5 Parallel Agents → Final Synthesis → Streamed to Browser
```

| Agent | MCP Server | Port | Data Source |
|-------|-----------|------|-------------|
| EPHT_Agent | mcp_server_epht.py | 8889 | CDC Environmental Public Health Tracking |
| OpenData_Agent | mcp_server_opendata.py | 8890 | CDC Open Data (COVID, disease surveillance) |
| HealthcareAccess_Agent | mcp_server_healthcare.py | 8891 | Healthcare.gov (insurance, providers) |
| MedlinePlus_Agent | mcp_server_medlineplus.py | 8892 | MedlinePlus Connect (patient education, ICD-10) |
| OpenFDA_Agent | mcp_server_openfda.py | 8893 | OpenFDA (FAERS adverse events, recalls, MAUDE) |

## Project Structure

```
├── fastapi_server.py           # Main API — AGENT_CONFIGS, /chat SSE endpoint
├── start_servers.py            # Starts all 5 MCP servers sequentially
├── frontend.html               # React UI — 5 agent columns, final report
├── mcp_server_*.py             # One per agent, binds to its port
├── tools/                      # Per-source API integrations
├── tests/                      # Integration tests
├── docs/                       # Supplementary docs
├── Dockerfile                  # Single image for all services
├── docker-compose.yml          # 5 MCP services + 1 API service
└── .env.example                # Template — copy to .env, add ANTHROPIC_API_KEY
```

## How to Run

```bash
# Docker (easiest)
cp .env.example .env && docker-compose up --build && open frontend.html

# Local
cp .env.example .env && uv sync
uv run start_servers.py     # Terminal 1 — ports 8889-8893
uv run fastapi_server.py    # Terminal 2 — port 8000
open frontend.html
```

Port conflict fix: `lsof -ti:8889 | xargs kill -9`

## Key Implementation Notes

- `fastapi_server.py` reads MCP server URLs from env vars (`MCP_EPHT_URL` etc.) with `localhost` defaults — docker-compose overrides these with Docker service hostnames
- `mcp_server_healthcare.py` imports from `tools/healthcare_gov.py` (not `_fixed`)
- Frontend is a single HTML file — React via CDN, no build step
- AI model used: `claude-3-5-sonnet-20241022` (in fastapi_server.py — can update to latest)
- Agent selection is keyword-based in `select_relevant_agents()` — not all 5 agents fire on every query

## Bugs Fixed

1. `start_servers.py` was missing `mcp_server_epht.py` (port 8889) — EPHT_Agent failed to connect
2. Frontend had 4 agent columns, backend had 5 — EPHT column added
3. OpenFDA and CDC Open Data shared the same React ref — each now has its own
