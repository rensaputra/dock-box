# PostgreSQL Docker Compose Setup

This directory contains a Docker Compose configuration for running PostgreSQL 16 in a containerized environment, intended for local testing with the provided schema and sample data.

## Overview

- **Image**: PostgreSQL 16 (official)
- **Port**: 5432 (localhost)
- **Default Database**: `myapp`
- **App User**: `appuser` / `apppassword`
- **Credentials**: Read from `.env` (see `.env.example`)
- **Init Scripts**: Runs `schema.sql` then `data.sql` on first startup
- **Persistence**: Data is stored in a named volume (`pgdata`)

## Quick Start

```bash
# Copy the example env file and adjust values if needed
cp .env.example .env

docker compose up -d
```

## Verification

Check if PostgreSQL is running and healthy:

```bash
docker compose ps
```

Connect and verify data was initialized:

```bash
psql -h 127.0.0.1 -U appuser -d myapp -c "SELECT * FROM LIL.ROOMS;"
# Password: apppassword
```

## Files

- `docker-compose.yml` - Main Docker Compose configuration
- `.env.example` - Template for database credentials (copy to `.env`)
- `schema.sql` - Schema definition (schema, tables, keys, index)
- `data.sql` - Sample data inserts

## Init Script Ordering

Scripts in `/docker-entrypoint-initdb.d/` run in **alphabetical** order. Since
`data.sql` would sort before `schema.sql`, the compose file mounts them with
numeric prefixes so the schema is created before the data is inserted. The
source files are left unchanged:

```yaml
- ./schema.sql:/docker-entrypoint-initdb.d/01-schema.sql:ro
- ./data.sql:/docker-entrypoint-initdb.d/02-data.sql:ro
```

## Customization

### Database Credentials

Edit `.env`:

```env
POSTGRES_DB=your_database_name
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_password
```

These values only take effect during first-time initialization (see gotchas below).

## ⚠️ Important Gotchas

### Init Scripts Only Run on First Startup

`schema.sql` and `data.sql` execute **only once**, when the data directory is
empty. Later `docker compose up` commands will **not** re-run them. The same
applies to credential changes in `.env` — editing `POSTGRES_USER` or
`POSTGRES_PASSWORD` does not update an existing database.

To apply changes to the schema, data, or credentials, you must wipe the volume
and recreate:

```bash
docker compose down -v
docker compose up -d
```

### Persistent Data

Unlike a throwaway container, this setup persists data in the `pgdata` named
volume, so data survives `docker compose down` and `up`.

- `docker compose down` keeps the volume (and your data).
- `docker compose down -v` removes the volume (wipes data, re-runs init scripts).

## Useful Commands

```bash
# Start container
docker compose up -d

# Stop container (keeps data)
docker compose down

# Stop and wipe data (re-runs init scripts next up)
docker compose down -v

# View logs
docker compose logs -f postgres

# Connect to PostgreSQL
psql -h 127.0.0.1 -U appuser -d myapp

# Execute query
psql -h 127.0.0.1 -U appuser -d myapp -c "\dt LIL.*"
```

## Troubleshooting

### Container won't start

Check logs:

```bash
docker compose logs postgres
```

### Health check failing

Give PostgreSQL a few seconds to become ready; `pg_isready` is polled on an
interval before the service reports healthy.

### Can't connect

- Verify container is running: `docker compose ps`
- Verify port 5432 is not in use: `lsof -i :5432`
- Verify `.env` exists and credentials match what you're connecting with
