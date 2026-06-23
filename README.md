# Actief Werkt! RSS Feed

RSS 0.91 feed that queries the [Carerix GraphQL API](https://api.carerix.io/graphql/v1/graphql) for active job publications and outputs them as XML.

## What it does

- Fetches all publications with medium **"web"** and **"betaald"**
- Filters by `publicationStart <= today` and `publicationEnd > today` (or empty)
- Returns **RSS 0.91 XML** with all vacancy details (title, company, location, salary, description, education, etc.)
- Caches the feed in memory for 1 hour

## Prerequisites

- **Node.js 18+** (zero npm dependencies — uses built-in `fetch`)
- **Carerix OAuth2 credentials** (client ID, client secret, token endpoint)

## Quick Start

```bash
cp .env.example .env
# Edit .env with your Carerix credentials

node server.js
# → http://localhost:3000/api/rss
```

## Deployment Options

### Vercel (serverless)

The `api/rss.js` and `vercel.json` files are ready for Vercel deployment. Add your 3 environment variables in the Vercel dashboard and deploy.

### Docker

```bash
docker build -t actiefwerkt-rss .
docker run -d -p 3000:3000 --env-file .env actiefwerkt-rss
```

### Docker Compose

```bash
cp .env.example .env    # fill in credentials
docker compose up -d
```

### VPS / Bare Metal

```bash
npm install -g pm2
pm2 start server.js --name actiefwerkt-rss
pm2 save && pm2 startup
```

See [INSTALL.md](INSTALL.md) for detailed instructions including systemd service setup and Nginx reverse proxy with SSL.

## Configuration

| Variable | Required | Default | Description |
|---|---|---|---|
| `CARERIX_CLIENT_ID` | Yes | — | OAuth2 client ID |
| `CARERIX_CLIENT_SECRET` | Yes | — | OAuth2 client secret |
| `CARERIX_TOKEN_ENDPOINT` | Yes | — | OAuth2 token URL |
| `PORT` | No | `3000` | HTTP port |
| `CACHE_TTL_SECONDS` | No | `3600` | Cache lifetime (seconds) |

## Endpoints

| Path | Description |
|---|---|
| `/api/rss` | RSS 0.91 XML feed |
| `/` | Same as `/api/rss` |
| `/health` | JSON health check |

## License

UNLICENSED — private use only.
