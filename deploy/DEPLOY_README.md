# Deploy — Deployment Automation

Scripts for deploying the Deal Desk Agent to Google Cloud. Two targets: Cloud Run + GCE (the main demo path) and Agent Engine (optional managed variant).

## What's in here

| File | Purpose |
|---|---|
| `deploy.sh` | Full Cloud Run + GCE deployment pipeline |
| `agent_engine_deploy.py` | Agent Engine deployment for the `agent_deploy/` variant |

## `deploy.sh` — what it does

Interactive bash script, runs 8 steps with colored output and a confirmation prompt:

1. **Enable APIs** — Vertex AI, BigQuery, Cloud Run, Compute, Cloud Build, Artifact Registry, Discovery Engine
2. **Create Artifact Registry repo** — `deal-desk-agent` in `us-central1`
3. **Create service account** — `deal-desk-agent-sa` with `roles/aiplatform.user`, `roles/bigquery.dataEditor`, `roles/bigquery.jobUser`
4. **Build backend container** — `gcloud builds submit`
5. **Build frontend container** — same, with Vite build baked in
6. **Build browser VM container** — the Computer Use image
7. **Deploy Cloud Run services** — backend and frontend with env vars
8. **Create GCE VM** — `deal-desk-browser` with static IP, firewall rule, network tag

### Environment variables

```bash
export PROJECT_ID="your-project"            # required
export REGION="us-east5"                    # Claude model region
export INFRA_REGION="us-central1"           # Cloud Run + GCE region
./deploy.sh
```

The script confirms the target project before doing anything destructive.

### Prerequisites

- `gcloud` CLI authenticated
- A `default` VPC network exists (or edit the script to use your VPC)
- Billing enabled on the project
- Claude models enabled in Vertex AI Model Garden for `us-east5`

## `agent_engine_deploy.py` — what it does

Python script that packages the `agent_deploy/` folder as a Vertex AI Agent Engine application. Uses `vertexai.preview.reasoning_engines.AdkApp` to upload and register the agent.

### Run

```bash
cd deploy
pip install google-cloud-aiplatform[agent_engines,adk]
python agent_engine_deploy.py
```

Returns the Agent Engine resource ID on success. Save this — it's needed for A2A registration with Gemini Enterprise.

## Cleanup

There's no `cleanup.sh` in this folder (yet). To tear down all resources manually:

```bash
gcloud run services delete deal-desk-backend deal-desk-frontend --region=us-central1 --quiet
gcloud compute instances delete deal-desk-browser --zone=us-central1-a --quiet
gcloud compute addresses delete deal-desk-browser-ip --region=us-central1 --quiet
gcloud compute firewall-rules delete deal-desk-browser-ports --quiet
bq rm -r -f -d $PROJECT_ID:deal_desk_agent
gcloud artifacts repositories delete deal-desk-agent --location=us-central1 --quiet
```

See the top-level `README.md` troubleshooting section for deployment failure modes.
