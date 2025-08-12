# Local development setup (Docker: Ghost + MySQL)

Run all commands from:
```
/Users/teeshaghevariya/Desktop/Mercedes-Prep/ghost-dev
```

## Prerequisites
- Docker Desktop installed and running
- Git configured with SSH

Verify Docker:
```bash
docker version
docker compose version
```

## 1) Fork, clone, and branch
```bash
cd /Users/teeshaghevariya/Desktop/Mercedes-Prep
# Fork TryGhost/Ghost on GitHub first, then clone your fork
git clone git@github.com:teeshaa/Ghost.git ghost-dev
cd ghost-dev
# Track upstream for future updates
git remote add upstream git@github.com:TryGhost/Ghost.git
# Work on a feature branch
git checkout -b feat/teesha-testing
```

## 2) Use the Ghost + MySQL compose file explicitly
This repository contains two Compose files:
- `compose.yml` → monorepo dev stack (builds the source, multiple services)
- `docker-compose.yml` → Ghost + MySQL (single Ghost container + external MySQL)

Shut down any monorepo stack if started:
```bash
COMPOSE_PROFILES=ghost docker compose down
```

## 3) Start Ghost + MySQL (pinned images)
```bash
docker compose -f docker-compose.yml up -d
```
`docker-compose.yml` configuration specifics:
- Image: `ghost:5.88.3-alpine`
- DB image: `mysql:8.0.36`
- Ports: `2368:2368` (Ghost), `3307:3306` (MySQL)
- Volume: `ghost_mysql_data`
- Mount: `./content:/var/lib/ghost/content`

## 4) Verify services and HTTP
```bash
docker compose -f docker-compose.yml ps
```
Expected mappings:
- `ghost` → `0.0.0.0:2368->2368/tcp`
- `db` → `0.0.0.0:3307->3306/tcp`

Probe HTTP:
```bash
curl -sS -I http://localhost:2368
```
`HTTP/1.1 200 OK` confirms Ghost is responding.

## 5) Open the app
- Site: `http://localhost:2368`
- Admin: `http://localhost:2368/ghost`

Complete the admin setup in the browser.

## 6) Logs and lifecycle
Follow Ghost logs:
```bash
docker compose -f docker-compose.yml logs -f ghost | cat
```
Restart Ghost only:
```bash
docker compose -f docker-compose.yml restart ghost
```
Stop stack (keep data):
```bash
docker compose -f docker-compose.yml down
```
Stop stack and remove data:
```bash
docker compose -f docker-compose.yml down -v
```

## 7) Theme/content development
Local `content` is mounted at `/var/lib/ghost/content` inside the container.
- Themes path: `content/themes/<your-theme>`
- Changes reflect immediately without rebuilding the container.

## 8) Troubleshooting
- Multiple compose files warning: always include `-f docker-compose.yml`.
- Port 2368 refused: verify with `ps`, then `curl -I http://localhost:2368`, then `restart ghost`.
- MySQL health: `docker compose -f docker-compose.yml logs -f db | cat` and confirm `STATUS` shows `(healthy)` in `ps`.
