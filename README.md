# Deal Desk Agent

**FSI Deal Desk Pipeline — Anthropic + Google Cloud: Better Together**

Built for Google Cloud NEXT 2026. A multi-agent system that automates FSI client onboarding using Claude models on Vertex AI, orchestrated by Google ADK.

## Architecture

User Prompt
  -> ParallelAgent (Research Agent [Opus 4.5] + Compliance Agent [Sonnet 4.6])
  -> Risk Scoring Agent [Haiku 4.5]
  -> Synthesis Agent [Opus 4.5]
  -> Salesforce Browser Agent [Sonnet 4.6, Computer Use API]

## Models

| Agent | Model | Vertex AI String | Role |
|-------|-------|-----------------|------|
| Research | Claude Opus 4.5 | claude-opus-4-5@20251101 | Client and market intelligence |
| Compliance | Claude Sonnet 4.6 | claude-sonnet-4-6@default | KYC/AML/sanctions checks |
| Risk | Claude Haiku 4.5 | claude-haiku-4-5@20251001 | Quantitative risk scoring |
| Synthesis | Claude Opus 4.5 | claude-opus-4-5@20251101 | Deal package assembly |
| Salesforce | Claude Sonnet 4.6 | claude-sonnet-4-6@default | Browser-based CRM entry |

Region: us-east5 | Project: cpe-slarbi-nvd-ant-demos

## GCP Services

Vertex AI, BigQuery, Cloud Run, Compute Engine, Artifact Registry, Agent Engine, ADK

## Project Structure

deal-desk-agent/
  backend/
    agents/ — ADK agent definitions, computer use loop, A2A agent card
    tools/ — BigQuery read/write tools, risk scoring engine
    main.py — FastAPI backend with SSE streaming
    Dockerfile, requirements.txt
  frontend/
    src/App.jsx — React command center UI
    Dockerfile, package.json, vite.config.js
  computer-use/
    Dockerfile — Browser VM (Xvfb + Chrome + noVNC)
    entrypoint.sh, supervisord.conf
  deploy/
    deploy.sh — Cloud Run deployment
    agent_engine_deploy.py — Agent Engine deployment
  docker-compose.yaml — Local development
  .env — Environment config

## Quick Start

### Local Development

  gcloud auth application-default login
  docker compose up --build

  Frontend:  http://localhost:3000
  Backend:   http://localhost:8080
  noVNC:     http://localhost:6080

### Deploy to GCP

  cd deploy && ./deploy.sh

### Deploy to Agent Engine

  cd deploy && python agent_engine_deploy.py

## Demo Runbook (Google NEXT Booth)

### Before the Conference
1. Deploy all services via deploy.sh
2. Deploy browser VM on GCE
3. Pre-authenticate Salesforce in the browser VM
4. Test the full flow end-to-end
5. Record a backup video of the demo

### Each Demo (3-5 minutes)
1. Click a preset scenario or type a custom prompt
2. Narrate as agents appear in the activity feed
3. Point out parallel execution (Research + Compliance)
4. Highlight the deal package summary
5. Watch the Salesforce agent drive the browser in real time
6. Click into Salesforce to verify the Opportunity

### Between Demos
- Click Reset to clean up demo data
- This deletes recent deal packages and resets client statuses

### If Something Breaks
- Backend down: Check Cloud Run logs
- Browser VM frozen: SSH into GCE and restart container
- Salesforce session expired: VNC into browser VM and re-login
- Nuclear option: Play the backup video

## Future Roadmap

- Gemini Enterprise: Set MODEL_PROVIDER=gemini in .env to swap all agents
- Agent Gallery: Register via Agent Engine for discovery
- GCP Marketplace: Package as a deployable solution
- LoopAgent: Add quality review loop around synthesis
- MCP Toolbox: Replace direct BigQuery client with MCP
