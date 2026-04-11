# Frontend — React + Vite Command Center UI

The React-based command center UI for the Deal Desk Agent. Streams real-time agent activity from the backend over SSE, displays the multi-agent pipeline's progress, and embeds the noVNC viewer for watching Claude drive Salesforce.

## What's in here

| File | Purpose |
|---|---|
| `src/App.jsx` | Main React app — chat UI, SSE consumer, agent activity feed |
| `src/Markdown.jsx` | Markdown rendering component for agent outputs |
| `src/main.jsx` | React entry point |
| `index.html` | Vite HTML template |
| `vite.config.js` | Vite build configuration |
| `package.json` | Dependencies and build scripts |
| `nginx.conf` | **Cloud Run-compatible nginx config (listens on port 8080)** |
| `Dockerfile` | Multi-stage build: Vite build → nginx:alpine serve |

## Critical: the nginx port fix

`nginx:alpine` listens on port 80 by default. Cloud Run probes `$PORT` (8080) and fails with `Default STARTUP TCP probe failed ... DEADLINE_EXCEEDED` if nginx isn't actually listening on 8080. The provided `nginx.conf` explicitly sets `listen 8080;` to fix this:

```nginx
server {
    listen 8080;
    root /usr/share/nginx/html;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**Do not delete or edit this file unless you know what you're doing.** It's the single most common deployment failure.

## Run locally

```bash
cd frontend
npm install
npm run dev
```

Dev server runs on http://localhost:3000 (Vite default). The app expects the backend at `http://localhost:8080` — override with `VITE_BACKEND_URL` env var if needed.

## Build and deploy

```bash
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/deal-desk-agent/frontend:latest
gcloud run deploy deal-desk-frontend --image us-central1-docker.pkg.dev/$PROJECT_ID/deal-desk-agent/frontend:latest --region us-central1
```

## SSE event types

The frontend consumes these event types from `/api/chat`:

- `tool_call` — backend is invoking a BigQuery tool
- `agent_start` — an ADK agent is activating (shown in the activity feed)
- `agent_output` — an agent emitted text output
- `pipeline_complete` — full pipeline finished
- `salesforce_started` — browser agent was triggered
- `chat_response` — final conversational response to display
