# CLAUDE.md

## Project overview

Docker packaging for [draw.io](https://github.com/jgraph/drawio). Produces the `jgraph/drawio` and `jgraph/export-server` Docker images.

## Repository structure

- `main/` — Primary draw.io Docker image (Tomcat + draw.io WAR built from source)
  - `Dockerfile` — Multi-stage build: builds draw.io WAR with Ant, deploys to Tomcat 9
  - `docker-entrypoint.sh` — Runtime configuration via environment variables (PreConfig.js / PostConfig.js generation, SSL setup, Tomcat context path)
- `image-export/` — Export server image (Node.js + Puppeteer + Chrome for PDF/image export)
- `self-contained/` — docker-compose for running draw.io with export server and all integrations
- `nextcloud/` — docker-compose for Nextcloud integration with nginx reverse proxy

## Key conventions

- The main branch is `dev`.
- Commit messages reference upstream issues as `[jgraph/drawio#NNN]` and docker-drawio issues as `[jgraph/docker-drawio#NNN]`.
- Environment variables are the primary configuration mechanism. They are processed in `main/docker-entrypoint.sh` which generates JS config files injected into the draw.io webapp at container startup.
- The upstream draw.io source is cloned at build time (not vendored). The WAR is built from `https://github.com/jgraph/drawio`.

## Building and testing

```bash
# Build the main image
docker build -t jgraph/drawio main/

# Build the export server image
docker build -t jgraph/export-server image-export/

# Run locally
docker run -it --rm -p 8080:8080 jgraph/drawio

# Run self-contained (with export server)
cd self-contained && docker-compose up
```

## Environment variable patterns

New environment variables should:
1. Be documented in `README.md` under "Environment variables"
2. Be processed in `main/docker-entrypoint.sh`
3. Follow the `DRAWIO_*` naming convention for draw.io-specific settings
4. Use shell defaults: `${VAR:-default_value}`
