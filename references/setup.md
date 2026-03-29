# Paperclip Setup Guide

## Prerequisites

- Paperclip CLI installed: `npm install -g paperclipai`
- Tailscale installed and connected (optional)
- GitHub CLI installed: `gh --version`
- GitHub CLI authenticated: `gh auth login`

---

## 1. Initialize Paperclip (local_trusted mode)

```bash
# Run onboarding with defaults
paperclipai onboard --yes

# Verify config
cat ~/.paperclip/instances/default/config.json | jq '.server'
```

Should show:
```json
{
  "deploymentMode": "local_trusted",
  "host": "127.0.0.1",
  "port": 3100
}
```

### If deploymentMode is not "local_trusted"

Edit config directly:

```bash
# Edit config
nano ~/.paperclip/instances/default/config.json

# Change:
# "deploymentMode": "authenticated" → "local_trusted"
# "host": "0.0.0.0" → "127.0.0.1"
```

---

## 2. Start Paperclip

```bash
# Start in background
paperclipai run &

# Verify
curl http://127.0.0.1:3100/api/health
# Should return: {"status":"ok",...}

# Test API (no auth needed in local_trusted)
curl http://127.0.0.1:3100/api/companies
# Should return: []
```

---

## 3. Setup Tailscale Serve Proxy (Optional)

**Optional:** This allows accessing Paperclip from any machine in your tailnet.

```bash
# Create proxy
tailscale serve --bg --http=3100 http://127.0.0.1:3100

# Verify
tailscale serve status | grep 3100
# Should show: 3100 -> proxy http://127.0.0.1:3100

# Test from another machine or via Tailscale URL
curl http://YOUR-MACHINE.tailXXXX.ts.net:3100/api/companies
# Should return: []
```

### Remove proxy

```bash
tailscale serve --http=3100 off
```

---

## 4. Verify Git Authentication

Required for automatic repo creation:

```bash
# Check if logged in
gh auth status

# Expected output:
# ✓ Logged in to github.com account YOUR_USERNAME
# ✓ Git operations for github.com configured to use https protocol
# ✓ Token: ghp_...
```

If not logged in:

```bash
gh auth login
# Choose: GitHub.com → HTTPS → Yes (authenticate Git) → Login with web browser
```

---

## 5. Verify Complete Setup

Run all checks:

```bash
# 1. Check deployment mode
cat ~/.paperclip/instances/default/config.json | jq '.server.deploymentMode'
# Expected: "local_trusted"

# 2. Check Paperclip health
curl -s http://127.0.0.1:3100/api/health | jq '.status'
# Expected: "ok"

# 3. Check git auth
gh auth status 2>&1 | grep "Logged in"
# Expected: "✓ Logged in to github.com account YOUR_USERNAME"
```

**Optional: Tailscale verification**

```bash
# Check Tailscale proxy (if configured)
tailscale serve status 2>/dev/null | grep 3100 || echo "No Tailscale proxy (optional)"
```

---

## 5. Keep Paperclip Running

### Option A: Systemd service (recommended)

```bash
# Create service file
sudo nano /etc/systemd/system/paperclip.service
```

```ini
[Unit]
Description=Paperclip AI
After=network.target

[Service]
Type=simple
User=YOUR_USER
WorkingDirectory=/home/YOUR_USER
ExecStart=/usr/bin/paperclipai run
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable paperclip
sudo systemctl start paperclip
```

### Option B: PM2

```bash
pm2 start "paperclipai run" --name paperclip
pm2 save
pm2 startup
```

### Option C: Screen/Tmux

```bash
screen -dmS paperclip paperclipai run
```

---

## Troubleshooting

### Port 3100 already in use

```bash
# Find what's using port
lsof -i :3100

# Kill process
kill -9 $(lsof -ti :3100)
```

### Tailscale proxy not working

```bash
# Check Tailscale status
tailscale status

# Remove and recreate proxy
tailscale serve --http=3100 off
tailscale serve --bg --http=3100 http://127.0.0.1:3100
```

### API returns "Board access required"

This means you're in `authenticated` mode, not `local_trusted`.

```bash
# Check mode
cat ~/.paperclip/instances/default/config.json | jq '.server.deploymentMode'

# If not local_trusted, edit config
nano ~/.paperclip/instances/default/config.json
# Change deploymentMode to "local_trusted"
# Change host to "127.0.0.1"

# Restart Paperclip
pkill -f "paperclipai run"
paperclipai run &
```

---

## Reference

| Setting | Value |
|---------|-------|
| Config | `~/.paperclip/instances/default/config.json` |
| Database | `~/.paperclip/instances/default/db` |
| Logs | `~/.paperclip/instances/default/logs` |
| Backups | `~/.paperclip/instances/default/data/backups` |
| API (local) | `http://127.0.0.1:3100/api` |
| UI (local) | `http://127.0.0.1:3100` |
