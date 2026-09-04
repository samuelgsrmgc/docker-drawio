# CLAUDE.md

## Project overview

Docker packaging for [draw.io](https://github.com/jgraph/drawio). Produces the `jgraph/drawio` Docker image.

The separate export server image (`jgraph/export-server`, formerly `image-export/`) is end-of-life and has been removed. Do not reintroduce `EXPORT_URL` / `DRAWIO_SELF_CONTAINED` handling or references to it. The one exception is the fixed `window.EXPORT_URL = null;` line the entrypoint writes into `PreConfig.js`: it tells the editor that no export service exists, so PDF export falls back to the print dialog instead of posting to convert.diagrams.net (which rejects other hosts). Keep that line; it is not export-server handling.

## Repository structure

- `main/` — The draw.io Docker image (Tomcat + draw.io WAR built from source)
  - `Dockerfile` — Multi-stage build: builds draw.io WAR with Ant, deploys to Tomcat 9
  - `docker-entrypoint.sh` — Runtime configuration via environment variables (PreConfig.js / PostConfig.js generation, SSL setup, Tomcat context path)
- `docker-compose.yml` — Runs the image with every `DRAWIO_*` variable passed through from the environment / a `.env` file (Google Drive, OneDrive and GitLab integrations)
- `nextcloud/` — docker-compose for Nextcloud integration with nginx reverse proxy
- `deploy/kubernetes/` — Kubernetes manifests
- `.github/workflows/docker-image-main.yml` — Builds, tests and pushes the image on version tags and weekly

## Key conventions

- The main branch is `dev`.
- Commit messages reference upstream issues as `[jgraph/drawio#NNN]` and docker-drawio issues as `[jgraph/docker-drawio#NNN]`.
- Environment variables are the primary configuration mechanism. They are processed in `main/docker-entrypoint.sh` which generates JS config files injected into the draw.io webapp at container startup.
- The upstream draw.io source is cloned at build time (not vendored). The WAR is built from `https://github.com/jgraph/drawio`.

## Building and testing

```bash
# Build the image
docker build -t jgraph/drawio main/

# Run locally
docker run -it --rm -p 8080:8080 jgraph/drawio

# Run with docker compose (configuration from a .env file)
docker compose up
```

## Environment variable patterns

New environment variables should:
1. Be documented in `README.md` under "Environment variables"
2. Be processed in `main/docker-entrypoint.sh`
3. Be passed through in `docker-compose.yml`
4. Follow the `DRAWIO_*` naming convention for draw.io-specific settings
5. Use shell defaults: `${VAR:-default_value}`
