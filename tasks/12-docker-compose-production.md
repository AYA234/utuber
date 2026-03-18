# Task 12 — Docker Compose Production Config

**Priority:** P2  
**Estimate:** 1–2 h  
**Assignee:** sw_developer  
**Depends on:** Task 01  
**Plan refs:** PLAN.md §4 (stack), §6 (docker-compose.yml), backlog #13

## Description
Extend the development `docker-compose.yml` with production-ready settings: health checks, restart policies, volume mounts, and resource limits.

## Acceptance Criteria
- Three services: `api`, `worker`, `redis`.
- Health checks on all three containers.
- `restart: unless-stopped` on `api` and `worker`.
- `TEMP_DOWNLOAD_DIR` mounted as a named volume shared between `api` and `worker`.
- `redis` data persisted via a named volume.
- No secrets in `docker-compose.yml` — all via `.env` file.
- `docker-compose up -d` starts the full stack cleanly from a cold start.

## Service Summary
| Service | Image | Ports |
|---|---|---|
| `api` | app Dockerfile | `8000:8000` |
| `worker` | app Dockerfile | — |
| `redis` | `redis:7-alpine` | internal only |

## 3rd-Party Tools
| Tool | Purpose |
|---|---|
| Docker Compose v2 | Orchestration |
| `redis:7-alpine` | Redis image |

## Notes
- Use a single `Dockerfile` for both `api` and `worker` containers — different `CMD` overrides.
- Consider a `docker-compose.override.yml` for local dev hot-reload (`uvicorn --reload`).
