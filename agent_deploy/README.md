# Agent Deploy — Vertex AI Agent Engine Variant

An alternative deployment target for the Deal Desk pipeline. Instead of running the multi-agent system in a Cloud Run FastAPI app, this folder packages it for **Vertex AI Agent Engine** — Google's fully managed runtime for ADK agents with built-in session management, scaling, and Memory Bank support.

## When to use this vs. `backend/`

| Scenario | Use |
|---|---|
| Conversational UI with SSE streaming, A2A endpoint, BigQuery reset, Salesforce browser trigger | `backend/` (Cloud Run) |
| Managed deployment, discovery via Agent Gallery, long-term memory | `agent_deploy/` (Agent Engine) |
| Both at once | Deploy both — the pipeline is the same, only the runtime differs |

## What's in here

| File | Purpose |
|---|---|
| `agent.py` | Agent Engine entry point — the `root_agent` exported for deployment |
| `tools.py` | BigQuery tools (Agent Engine variant) |
| `risk_scoring.py` | Risk scoring engine |
| `requirements.txt` | Agent Engine-specific dependencies |
| `__init__.py` | Package marker (required for relative imports) |

## Import structure

Unlike `backend/`, this folder **requires** `__init__.py` because it uses relative imports (`from .tools import ...`, `from .risk_scoring import ...`). Agent Engine's deployment tool packages the folder as a Python package, so the imports must be relative.

This is the opposite convention from `backend/` — don't mix them up.

## Environment variables

- `PROJECT_ID`
- `MODEL_REGION=us-east5` (must be force-set, not `setdefault`)
- `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION`

## Deploy

From the repo root:

```bash
cd deploy && python agent_engine_deploy.py
```

This uses `vertexai.preview.reasoning_engines.AdkApp` to package `agent_deploy/` and upload it to Agent Engine.

## Known constraints

- **No outbound HTTP to raw public IPs** — Agent Engine's sandbox blocks direct calls to the browser VM's static IP. Route through the Cloud Run backend's `/api/trigger-sf` endpoint instead.
- **`GOOGLE_CLOUD_LOCATION`** must be force-set before ADK imports, not via `os.environ.setdefault`.
- **Sonnet 4.6 model string** is `@default`, not a dated version.
