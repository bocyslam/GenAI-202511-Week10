# n8n Docker Setup Guide

This guide explains how to set up and run n8n with PostgreSQL using Docker Compose.

## Prerequisites

- Docker Desktop (or Docker Engine)
- Docker Compose (v1.29 or newer)
- Git (for cloning the repository)

## Quick Start

### 1. Clone the repository
```bash
git clone <your-repo-url>
cd GenAI-202511-Week10
```

### 2. Configure environment variables
Copy the example environment file and update values if needed:
```bash
cp .env.example .env
```

The `.env` file contains default development values that are ready to use. For production, update:
- `DB_PASSWORD` - Use a strong, unique password
- `N8N_WEBHOOK_URL` - Point to your actual n8n URL
- `N8N_PROTOCOL` - Change to `https` for production
- `N8N_SECURE_COOKIE` - Set to `true` for production

### 3. Start the containers
```bash
docker-compose up -d
```

This command will:
- Pull the latest n8n and PostgreSQL images
- Create Docker volumes for data persistence
- Start PostgreSQL with health checks
- Start n8n (waiting for PostgreSQL to be ready)
- Expose n8n on http://localhost:3333

### 4. Access n8n
Open your browser and navigate to:
```
http://localhost:3333
```

On first access, you'll be prompted to set up your n8n account.

## Docker Compose Services

### PostgreSQL Database
- **Container name**: n8n-postgres
- **Port**: 5432 (accessible from localhost)
- **Volumes**: `postgres_data` - persistent database storage
- **Health check**: Enabled - n8n will wait for the database to be ready

### n8n
- **Container name**: n8n
- **Port**: 3333
- **Volumes**: `n8n_data` - persistent n8n workflows and data
- **Depends on**: PostgreSQL (with health check)
- **Restart policy**: Unless stopped

## Common Commands

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f postgres
```

### Stop containers
```bash
docker-compose down
```

### Stop and remove volumes (WARNING: deletes all data)
```bash
docker-compose down -v
```

### Restart services
```bash
docker-compose restart
```

### Access PostgreSQL
```bash
docker exec -it n8n-postgres psql -U n8n_user -d n8n_db
```

## Troubleshooting

### n8n won't start or keeps restarting
1. Check logs: `docker-compose logs n8n`
2. Ensure PostgreSQL is healthy: `docker-compose logs postgres`
3. Verify environment variables in `.env` file
4. Check port 3333 is not already in use: `lsof -i :3333`

### Connection refused on port 3333
1. Ensure containers are running: `docker-compose ps`
2. Check n8n logs for errors: `docker-compose logs n8n`
3. Wait a few seconds - n8n may still be initializing

### Database connection errors
1. Verify PostgreSQL is healthy: `docker-compose ps`
2. Check database credentials in `.env` match docker-compose.yml
3. Review PostgreSQL logs: `docker-compose logs postgres`

### Port already in use
If port 3333 or 5432 is already in use:
1. Change the port in docker-compose.yml
2. Or stop the existing service using that port

## Environment Variables

See `.env.example` for a complete list of available configuration options.

### Key Variables:
- **DB_USER/DB_PASSWORD/DB_NAME** - PostgreSQL credentials
- **N8N_HOST/N8N_PORT** - n8n server address
- **N8N_PROTOCOL** - http or https
- **N8N_WEBHOOK_URL** - External URL for webhooks
- **N8N_LOG_LEVEL** - Logging verbosity (debug, info, warn, error)

## Data Persistence

Both n8n and PostgreSQL use named Docker volumes:
- `postgres_data` - Database files
- `n8n_data` - n8n configuration and workflows

These volumes persist even when containers are stopped. To reset everything, use:
```bash
docker-compose down -v
```

## Production Considerations

Before deploying to production:

1. **Update environment variables** in `.env` with production values
2. **Use strong passwords** for database access
3. **Enable HTTPS** - Set `N8N_PROTOCOL=https` and `N8N_SECURE_COOKIE=true`
4. **Set proper webhook URL** - Should match your production domain
5. **Use environment-specific secrets** - Never commit real credentials
6. **Configure backups** - Regularly backup PostgreSQL volumes
7. **Use a reverse proxy** (Nginx/Traefik) - Handle SSL termination
8. **Monitor container health** - Set up logging and alerting
9. **Regular updates** - Keep Docker images updated

## Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- [n8n Docker Hub](https://hub.docker.com/r/n8nio/n8n)
