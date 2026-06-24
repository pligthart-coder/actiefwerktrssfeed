# Installation Guide — Self-Hosted Deployment

This guide explains how to deploy the Actief Werkt! RSS feed server on your own infrastructure.

## Prerequisites

- **Node.js 18+** (uses built-in `fetch`, zero npm dependencies)
- **Carerix OAuth2 credentials** (client ID, client secret, token endpoint)

---

## Step 1: Get the code

```bash
git clone https://github.com/pligthart-coder/actiefwerktrssfeed.git
cd actiefwerktrssfeed
```

## Step 2: Configure credentials

```bash
cp .env.example .env
```

Edit `.env` and fill in your Carerix credentials:

```env
CARERIX_CLIENT_ID=your-client-id
CARERIX_CLIENT_SECRET=your-client-secret
CARERIX_TOKEN_ENDPOINT=https://yourcompany.carerix.com/cxoauth2/token
```

You can obtain these from **Carerix → Identity Access → Clients**.

## Step 3: Start the server

```bash
node server.js
```

The feed will be available at `http://localhost:3000/api/rss`.

---

## Deployment Options

### Option 1: Docker Compose (recommended)

```bash
cp .env.example .env    # fill in credentials
docker compose up -d

# View logs
docker compose logs -f

# Stop
docker compose down
```

### Option 2: Docker (manual)

```bash
# Build
docker build -t actiefwerkt-rss .

# Run
docker run -d \
  --name actiefwerkt-rss \
  --restart unless-stopped \
  -p 3000:3000 \
  --env-file .env \
  actiefwerkt-rss
```

### Option 3: PM2 (process manager)

```bash
# Install PM2
npm install -g pm2

# Start the server
pm2 start server.js --name actiefwerkt-rss

# Save and auto-start on boot
pm2 save
pm2 startup
```

### Option 4: systemd service (Linux)

Create `/etc/systemd/system/actiefwerkt-rss.service`:

```ini
[Unit]
Description=Actief Werkt RSS Feed Server
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/actiefwerkt-rss
EnvironmentFile=/opt/actiefwerkt-rss/.env
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Then:

```bash
# Copy files to /opt
sudo mkdir -p /opt/actiefwerkt-rss
sudo cp server.js package.json /opt/actiefwerkt-rss/
sudo cp .env.example /opt/actiefwerkt-rss/.env
sudo nano /opt/actiefwerkt-rss/.env  # fill in credentials

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable actiefwerkt-rss
sudo systemctl start actiefwerkt-rss

# Check status
sudo systemctl status actiefwerkt-rss
journalctl -u actiefwerkt-rss -f
```

---

## Reverse Proxy with Nginx + SSL

To serve the feed on port 80/443:

```nginx
server {
    listen 80;
    server_name rss.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 120s;
    }
}
```

Add SSL with Let's Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d rss.yourdomain.com
```

---

## Endpoints

| Path | Description |
|---|---|
| `/api/rss` | Full RSS feed (all publications) |
| `/api/rss?medium=web` | Only "web" publications |
| `/api/rss?medium=betaald` | Only "betaald" publications |
| `/health` | JSON health check (`{"status":"ok","cached":true,"cacheAge":123}`) |

---

## Notes

- **Zero npm dependencies** — uses only Node.js built-in modules and the native `fetch` API.
- **In-memory cache** — the feed is cached for 1 hour by default. The first request after startup takes ~30-50 seconds while it fetches all publications from the Carerix API.
- **Split feeds** — using `?medium=web` or `?medium=betaald` returns roughly half the data, improving response times.
