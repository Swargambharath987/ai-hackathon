# HealthGuard AI — Project Context

## Overview
AI-powered parallel health data research platform for public health officials.
**5 specialized agents** query CDC, FDA, MedlinePlus, and Healthcare.gov simultaneously, then synthesize results into a comprehensive report.

## Repo
- **GitHub:** https://github.com/Swargambharath987/ai-hackathon
- **Working branch:** `bharath` (default)
- **Original upstream:** `JZolton/ai-hackathon` (forked Jun 2025)

## Architecture

```
User Query → FastAPI (/chat SSE) → 5 Parallel Agents → Final Synthesis → Streamed to Frontend
```

| Agent | MCP Server | Port | Data Source |
|-------|-----------|------|-------------|
| EPHT_Agent | mcp_server_epht.py | 8889 | CDC Environmental Public Health Tracking |
| OpenData_Agent | mcp_server_opendata.py | 8890 | CDC Open Data (COVID, disease surveillance) |
| HealthcareAccess_Agent | mcp_server_healthcare.py | 8891 | Healthcare.gov (insurance, providers) |
| MedlinePlus_Agent | mcp_server_medlineplus.py | 8892 | MedlinePlus Connect (patient education, ICD-10) |
| OpenFDA_Agent | mcp_server_openfda.py | 8893 | OpenFDA (FAERS adverse events, recalls, MAUDE) |

**Backend:** FastAPI + Server-Sent Events streaming
**Frontend:** Single HTML file (`frontend_example.html`) — React via CDN, no build step
**AI:** Claude claude-sonnet-4-6 via `autogen-ext[anthropic]` + MCP tool protocol
**Package manager:** `uv` (pyproject.toml)

## Key Files

| File | Purpose |
|------|---------|
| `fastapi_server.py` | Main API — defines AGENT_CONFIGS, streaming /chat endpoint |
| `start_servers.py` | Starts all 5 MCP servers sequentially |
| `frontend_example.html` | Full React UI — 5 agent columns + final report |
| `parallel_agent.py` | Core parallel agent execution logic |
| `tools/` | Per-source API integrations (imported by MCP servers) |
| `.env.example` | Template — copy to `.env` and add ANTHROPIC_API_KEY |
| `pyproject.toml` | Dependencies (fastmcp, autogen-agentchat, autogen-ext, fastapi, tiktoken) |

## How to Run

```bash
# 1. Setup (once)
cp .env.example .env          # add your ANTHROPIC_API_KEY
uv sync

# 2. Terminal 1 — start all 5 MCP servers
uv run start_servers.py

# 3. Terminal 2 — start FastAPI backend
uv run fastapi_server.py      # runs on http://localhost:8000

# 4. Open frontend
open frontend_example.html    # or double-click in Finder
```

Port check if something won't start:
```bash
lsof -ti:8889 | xargs kill -9   # replace port as needed
```

## Bugs Fixed (Feb 2026)

1. **`start_servers.py` was missing EPHT** — `mcp_server_epht.py` (port 8889) was never started, causing EPHT_Agent to fail on connect. Fixed by adding it to the servers list.

2. **Frontend had 4 agents, backend had 5** — `EPHT_Agent` existed in `fastapi_server.py` but had no column in `frontend_example.html`. Fixed by adding the `🌍 EPHT` column and `EPHT_Agent` state.

3. **Duplicate `openDataRef`** — OpenFDA and CDC Open Data columns both used the same React ref, breaking auto-scroll. Fixed by giving each column its own ref (`ephtRef`, `medlineplusRef`, `openfdaRef`, `openDataRef`, `healthcareRef`).

## Branch History
- `main` — single working branch: 5 agents, all bugs fixed, full docs
