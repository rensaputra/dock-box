# MySQL Docker Compose Setup

This directory contains a Docker Compose configuration for running MySQL 8.0 in a containerized environment. It's designed to help you migrate from a direct MySQL installation to Docker.

## Overview

- **Image**: MySQL 8.0 (official)
- **Port**: 3306 (localhost)
- **Default Database**: `myapp`
- **Root Password**: `root`
- **App User**: `appuser` / `apppassword`
- **Init Scripts**: Automatically runs `init.sql` on first startup

## Quick Start

```bash
docker-compose up -d
```

## Verification

Check if MySQL is running and healthy:

```bash
docker-compose ps
```

Connect and verify data was initialized:

```bash
mysql -h 127.0.0.1 -u appuser -p -e "SELECT * FROM myapp.users;"
# Password: apppassword
```

## Files

- `docker-compose.yml` - Main Docker Compose configuration
- `init.sql` - SQL initialization script with database, table, and sample data

## Customization

### Database Credentials

Edit the `environment` section in `docker-compose.yml`:

```yaml
MYSQL_ROOT_PASSWORD: your_root_password
MYSQL_DATABASE: your_database_name
MYSQL_USER: your_username
MYSQL_PASSWORD: your_password
```

### Initialization Data

Edit `init.sql` to create your custom schema and insert your data:

```sql
CREATE DATABASE IF NOT EXISTS myapp;
USE myapp;

CREATE TABLE IF NOT EXISTS your_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

INSERT INTO your_table (name) VALUES ('value1'), ('value2');
```

## ⚠️ Important Gotchas

### Init Scripts Only Run on First Startup

The `init.sql` file is executed **only once** when the container is created for the first time. Subsequent `docker-compose up` commands will **not** re-run the script.

**If you modify `init.sql` and want to apply changes**, you must destroy the container and recreate it:

```bash
# Stop and remove the container
docker-compose down

# Optional: Clean up volumes if you want a fresh database
docker volume prune

# Recreate and start
docker-compose up -d
```

**WARNING**: Running `docker-compose down` will delete all data in the container. Make sure to back up any important data first.

### Persistent Data

By default, this setup does **not** persist data to your local filesystem. When you run `docker-compose down`, all database changes are lost.

To enable persistent storage, add a `volumes` section to `docker-compose.yml`:

```yaml
volumes:
  - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

Then data will survive container restarts.

## Useful Commands

```bash
# Start container
docker-compose up -d

# Stop container
docker-compose down

# View logs
docker-compose logs -f mysql

# Connect to MySQL
mysql -h 127.0.0.1 -u appuser -p

# Execute query
mysql -h 127.0.0.1 -u appuser -p -e "SHOW DATABASES;"

# Reset everything (caution: deletes data)
docker-compose down && docker volume prune && docker-compose up -d
```

## Troubleshooting

### Container won't start

Check logs:

```bash
docker-compose logs mysql
```

### Health check failing

Wait a bit longer for MySQL to be ready (usually 10-20 seconds).

### Can't connect

- Verify container is running: `docker-compose ps`
- Verify port 3306 is not in use: `lsof -i :3306`
- Check credentials match `docker-compose.yml`
