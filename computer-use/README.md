# Computer Use — Browser VM for Salesforce Automation

A Docker container that runs on a GCE VM, hosting Chrome + Xvfb + noVNC + a FastAPI server. Claude uses Computer Use (via Vertex AI `rawPredict`) to take screenshots, click, type, and navigate real Salesforce Lightning to create Opportunities.

## What's in here

| File | Purpose |
|---|---|
| `agent_server.py` | FastAPI server with the screenshot-reason-act Computer Use loop |
| `salesforce_browser_agent.py` | Salesforce-specific system prompt and helpers |
| `entrypoint.sh` | Container startup — launches supervisord |
| `supervisord.conf` | Process supervision (keeps Chrome alive on crash) |
| `Dockerfile` | Ubuntu 22.04 base with Chrome, Xvfb, noVNC, xdotool |

## Stack

| Component | Port | Purpose |
|---|---|---|
| Xvfb | display `:1` | Virtual X display (1280x800) |
| Fluxbox | — | Window manager |
| x11vnc | 5900 | VNC server backing the X display |
| noVNC + websockify | 6080 | Web-based VNC client |
| `agent_server.py` (FastAPI) | 8090 | Receives deal packages, runs Computer Use loop |

All four services are managed by **supervisord** with `autorestart=true`. If Chrome crashes during a demo, supervisord restarts it within seconds — this was added after a live demo where Chrome died and left the noVNC tab showing the empty Fluxbox desktop.

## Computer Use on Vertex AI — critical constraint

The tool type is `"custom"` with a full JSON Schema, **not** `computer_20250124`. `anthropic-beta` headers are **not supported** on Vertex AI `rawPredict`. See `agent_server.py` for the correct tool definition.

## Build and deploy

```bash
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/deal-desk-agent/browser-vm:latest
gcloud compute instances create-with-container deal-desk-browser \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --boot-disk-size=30GB \
  --container-image=us-central1-docker.pkg.dev/$PROJECT_ID/deal-desk-agent/browser-vm:latest \
  --tags=deal-desk-browser \
  --network=default
```

Then reserve a static IP, open the firewall for ports 6080 and 8090, and update the backend's `BROWSER_AGENT_URL`.

## First-time Salesforce login

Open `http://<STATIC_IP>:6080/vnc.html?autoconnect=true` in a browser. **Expect a device verification email** — Salesforce challenges every login from a new IP. Check your SF Dev Edition email, paste the code, and leave the browser on the Lightning home page. Subsequent demos reuse the session.

## Troubleshooting

- **Black screen in noVNC** — Xvfb hasn't started yet; wait 30 seconds
- **Empty Fluxbox desktop (no Chrome)** — supervisord should auto-restart Chrome; if not, `docker exec` into the container and check `/var/log/supervisord.log`
- **`computer_20250124 is not allowed`** — wrong tool type, use `"custom"`
- **Salesforce kicked us out** — re-login via noVNC, device verification may fire again
