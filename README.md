# Actief Werkt! RSS Feed

RSS 0.91 feed server that queries the [Carerix GraphQL API](https://api.carerix.io/graphql/v1/graphql) for active job publications and outputs them as XML.

## What it does

- Fetches publications from Carerix with medium **"web"** and/or **"betaald"**
- Filters by `publicationStart <= today` and `publicationEnd > today` (or empty)
- Returns **RSS 0.91 XML** with all vacancy details (title, company, location, salary, description, education, etc.)
- Caches the feed in memory for 1 hour (configurable)
- Supports filtering by medium via query parameter

## Prerequisites

- **Node.js 18+** (zero npm dependencies — uses built-in `fetch`)
- **Carerix OAuth2 credentials** (client ID, client secret, token endpoint)

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/pligthart-coder/actiefwerktrssfeed.git
cd actiefwerktrssfeed

# 2. Configure credentials
cp .env.example .env
# Edit .env with your Carerix credentials

# 3. Start the server
node server.js
```

The feed is available at `http://localhost:3000/api/rss`.

## Endpoints

| Path | Description |
|---|---|
| `/api/rss` | Full RSS feed (all publications) |
| `/api/rss?medium=web` | Only publications with medium "web" |
| `/api/rss?medium=betaald` | Only publications with medium "betaald" |
| `/health` | JSON health check |

## Configuration

| Variable | Required | Default | Description |
|---|---|---|---|
| `CARERIX_CLIENT_ID` | Yes | — | OAuth2 client ID |
| `CARERIX_CLIENT_SECRET` | Yes | — | OAuth2 client secret |
| `CARERIX_TOKEN_ENDPOINT` | Yes | — | OAuth2 token URL |
| `PORT` | No | `3000` | HTTP port |
| `CACHE_TTL_SECONDS` | No | `3600` | Cache lifetime in seconds |

## Deployment Options

### Docker (recommended)

```bash
cp .env.example .env    # fill in credentials
docker compose up -d
```

### Docker (manual)

```bash
docker build -t actiefwerkt-rss .
docker run -d --restart unless-stopped -p 3000:3000 --env-file .env actiefwerkt-rss
```

### PM2 (process manager)

```bash
npm install -g pm2
pm2 start server.js --name actiefwerkt-rss
pm2 save && pm2 startup
```

### systemd

See [INSTALL.md](INSTALL.md) for detailed systemd service setup and Nginx reverse proxy with SSL.

## License

UNLICENSED — private use only.
