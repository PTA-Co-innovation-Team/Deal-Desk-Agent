# Backend — FastAPI + ADK Multi-Agent Pipeline

The Deal Desk Agent backend. A FastAPI application that serves the ADK multi-agent pipeline with Server-Sent Events (SSE) streaming for real-time agent activity, plus an A2A endpoint for Gemini Enterprise integration.

## What's in here

| File | Purpose |
|---|---|
| `main.py` | FastAPI app — `/api/chat` (SSE), `/` (A2A), `/api/health`, `/api/reset` |
| `agents/deal_desk_swarm.py` | ADK pipeline: `ParallelAgent` + `SequentialAgent` with 5 Claude agents |
| `agents/salesforce_browser_agent.py` | HTTP client that triggers the Computer Use browser agent |
| `agents/agent_card.json` | A2A protocol agent card for Gemini Enterprise registration |
| `tools/bigquery_tools.py` | BigQuery read/write tools for all agents |
| `tools/risk_scoring.py` | Quantitative risk scoring engine |
| `Dockerfile` | Python 3.11-slim, uvicorn on port 8080 |
| `requirements.txt` | Pinned dependencies |

## Architecture

`main.py` wires an ADK `Runner` with `InMemorySessionService` (or `VertexAiSessionService` for Agent Engine) to the `deal_desk_pipeline` imported from `agents/`. The pipeline is a `SequentialAgent` containing a `ParallelAgent` (Research + Compliance) followed by Risk and Synthesis agents. The final Salesforce step is triggered via HTTP to the browser VM.

## Environment variables

See `.env.example` at the repo root. Required:

- `PROJECT_ID` — Google Cloud project ID
- `MODEL_REGION` — must be `us-east5` (Claude model region)
- `BROWSER_AGENT_URL` — `http://<BROWSER_VM_IP>:8090` for Salesforce automation
- `OPUS_MODEL`, `SONNET_MODEL`, `HAIKU_MODEL` — Vertex AI model strings
- `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION` — required by ADK

## Run locally

```bash
cd backend
pip install -r requirements.txt
export PROJECT_ID=your-project
export MODEL_REGION=us-east5
export GOOGLE_CLOUD_PROJECT=$PROJECT_ID
export GOOGLE_CLOUD_LOCATION=$MODEL_REGION
uvicorn main:app --host 0.0.0.0 --port 8080 --reload
```

## Build and deploy to Cloud Run

```bash
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/deal-desk-agent/backend:latest
gcloud run deploy deal-desk-backend --image us-central1-docker.pkg.dev/$PROJECT_ID/deal-desk-agent/backend:latest --region us-central1
```

Or use `deploy/deploy.sh` from the repo root for the full pipeline.

## Import structure gotcha

Backend uses **flat absolute imports** (`from agents import deal_desk_pipeline`, `from tools.bigquery_tools import ...`). Do **not** add an `__init__.py` at the top of `backend/` — the Cloud Run Dockerfile runs `uvicorn main:app` directly with `WORKDIR /app`, which expects modules importable by bare name. Adding `__init__.py` causes `ImportError: attempted relative import with no known parent package`.

## See also

- Top-level `README.md` for architecture overview and the full model table
- `agent_deploy/README.md` for the Agent Engine variant
