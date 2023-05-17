# Store.londonparkour.com

This repository contains the infrastructure and code to fully deploy the store.londonparkour.com website.

## docker-compose

### Starting the containers

```bash
docker compose up -d
```

### Switching from DEV to LIVE

The `.env` file has a `COMPOSE_PROFILES=dev` variable. Switch to `COMPOSE_PROFILES=live`

