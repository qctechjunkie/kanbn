# kan.bn — Self-Hosted Docker Compose

Self-hosted deployment of [kan.bn](https://kan.bn) using Docker Compose.

Official documentation: https://docs.kan.bn/guides/self-hosting/introduction

## Prerequisites

- Docker and Docker Compose installed
- A domain or IP address reachable by your users
- A 32+ character random string for `BETTER_AUTH_SECRET` (see below)

## Quick Start

**1. Copy the example environment file and fill in the required values:**

```bash
cp .env.example .env
```

Edit `.env` and set at minimum:

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_BASE_URL` | Public URL of your instance, e.g. `https://kan.example.com` |
| `BETTER_AUTH_SECRET` | 32+ character random string — generate with `openssl rand -base64 32` |
| `POSTGRES_PASSWORD` | Password for the PostgreSQL user |
| `POSTGRES_URL` | Full connection string — update the password to match `POSTGRES_PASSWORD` |

**2. Run database migrations (required on first launch and after upgrades):**

```bash
docker compose --profile migrate run --rm migrate
```

**3. Start the application:**

```bash
docker compose up -d
```

The web interface is available at the port defined by `WEB_PORT` (default: `http://localhost:3000`).

## Environment Variables

### Required

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_BASE_URL` | Public URL of this instance. No trailing slash. |
| `BETTER_AUTH_SECRET` | Secret key for signing auth sessions. Must be 32+ random characters. Changing this invalidates all active sessions. |
| `POSTGRES_URL` | PostgreSQL connection string. The hostname must be `postgres` (the Compose service name). Format: `postgresql://kan:<password>@postgres:5432/kan_db` |
| `POSTGRES_PASSWORD` | PostgreSQL password. Must match the password in `POSTGRES_URL`. |

### Optional

| Variable | Default | Description |
|---|---|---|
| `POSTGRES_USER` | `kan` | PostgreSQL username. Must match `POSTGRES_URL`. |
| `POSTGRES_DB` | `kan_db` | PostgreSQL database name. Must match `POSTGRES_URL`. |
| `WEB_PORT` | `3000` | Host port the web app is exposed on. |
| `DOCKER_DATA_PATH` | `./kan_data` | Host path for persistent PostgreSQL data. |
| `NEXT_PUBLIC_DISABLE_SIGN_UP` | `false` | Set to `true` to prevent new user registrations. |

#### Email / SMTP

Required for password reset emails and invitations. All six variables must be set together; the section is commented out in `.env.example` by default.

| Variable | Description |
|---|---|
| `EMAIL_FROM` | Sender address, e.g. `Kan <noreply@example.com>` |
| `SMTP_HOST` | SMTP server hostname |
| `SMTP_PORT` | SMTP port (typically `587` for STARTTLS, `465` for TLS) |
| `SMTP_USER` | SMTP username |
| `SMTP_PASSWORD` | SMTP password |
| `SMTP_SECURE` | `true` to require TLS; `false` for STARTTLS or plain text |

#### S3-Compatible File Storage

Required for avatar and attachment uploads. The section is commented out in `.env.example` by default.

| Variable | Description |
|---|---|
| `S3_ENDPOINT` | S3 endpoint URL |
| `S3_ACCESS_KEY_ID` | S3 access key ID |
| `S3_SECRET_ACCESS_KEY` | S3 secret access key |
| `NEXT_PUBLIC_STORAGE_URL` | Public URL for stored files |
| `NEXT_PUBLIC_STORAGE_DOMAIN` | Domain used for storage (for CSP headers) |

#### Integrations

| Variable | Description |
|---|---|
| `TRELLO_APP_API_KEY` | Trello API key for board import |
| `TRELLO_APP_SECRET` | Trello API secret |

## Common Commands

```bash
# Start all services in the background
docker compose up -d

# Stop all services
docker compose down

# View live logs
docker compose logs -f

# View logs for a specific service
docker compose logs -f web

# Restart the web service
docker compose restart web

# Run database migrations (first launch and after upgrades)
docker compose --profile migrate run --rm migrate

# Pull the latest images
docker compose pull

# Upgrade: pull new images, run migrations, then restart
docker compose pull && \
docker compose --profile migrate run --rm migrate && \
docker compose up -d
```

## Data Persistence

PostgreSQL data is stored at the path set by `DOCKER_DATA_PATH` (defaults to `./kan_data/db` relative to the Compose file). Back up this directory to preserve your data.

## Notes

- The PostgreSQL port (`5432`) is not exposed to the host by default. All communication between services happens over the internal Docker network. Uncomment the `ports` block in `compose.yaml` only if you need external database access (e.g. during local development with a GUI client).
- `NEXT_PUBLIC_ALLOW_CREDENTIALS=true` is hardcoded in `compose.yaml` and does not need to appear in `.env`.
- Optional variables in `.env.example` are commented out by default. Leave them commented out — rather than setting them to an empty string — to disable the associated feature. The app's environment validation rejects empty strings for URL and credential fields and will refuse to start.
