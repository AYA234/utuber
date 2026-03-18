# Task 01 — Project Scaffolding & Docker Setup

**Priority:** P0  
**Estimate:** 2–3 h  
**Assignee:** sw_developer  
**Plan refs:** PLAN.md §4 (stack), §6 (project structure), §9 (.env.example), backlog #1

## Description
Create the full repository skeleton: folder layout, FastAPI entry-point, Celery worker entry-point, Redis connection, and Docker Compose so the entire stack starts with one command.

## Acceptance Criteria
- Folder structure matches PLAN.md §6 exactly.
- `docker-compose up` starts three containers: `api`, `worker`, `redis`.
- `GET /` returns `{"status": "ok"}`.
- Celery worker connects to Redis and logs `ready`.
- `requirements.txt` pins all dependencies.
- `.env.example` matches PLAN.md §9.

## Deliverables
```
utuber/
├── requirements.txt
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── config/netfree_policy.yaml      # empty placeholder
└── src/
    ├── __init__.py
    ├── main.py                     # FastAPI app + health route
    ├── config.py                   # pydantic-settings Settings class
    ├── models.py                   # placeholder Pydantic models
    ├── api/__init__.py
    ├── services/__init__.py
    ├── workers/__init__.py
    └── templates/                  # empty dir
```

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `fastapi` | Web framework |
| `uvicorn` | ASGI server |
| `celery[redis]` | Task queue + Redis broker |
| `redis` | Redis client |
| `pydantic-settings` | Env-based config |
| `python-dotenv` | Load `.env` file |
| Docker / Docker Compose | Container orchestration |

## Notes
- Python 3.12+.
- All secrets via environment variables — nothing hardcoded.
- `TEMP_DOWNLOAD_DIR` should be created at startup if it doesn't exist.
