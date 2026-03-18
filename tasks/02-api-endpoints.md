# Task 02 — API Endpoints

**Priority:** P0  
**Estimate:** 2 h  
**Assignee:** sw_developer  
**Depends on:** Task 01  
**Plan refs:** PLAN.md §7 (API contract), §3 (system flow), §8.4 (worker task), backlog #2

## Description
Implement the two REST endpoints in `src/api/routes.py`:
- `POST /request` — accept URL + email, enqueue Celery task, return `request_id`.
- `GET /status/{request_id}` — read status from Redis, return JSON.

## Acceptance Criteria
- `POST /request` with valid body returns `202` with `{request_id, status: "queued", message}`.
- `POST /request` with missing/invalid fields returns `422`.
- `GET /status/{id}` returns the correct status object from Redis.
- `GET /status/{unknown_id}` returns `404`.
- Request models validated with Pydantic (see `src/models.py`).

## Request / Response Shapes
See PLAN.md §7 for exact JSON schemas.

**Status values:** `queued | analyzing | downloading | sending | completed | rejected | failed`

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `fastapi` | Router, request/response models |
| `pydantic` | Input validation |
| `redis` | Status reads |
| `celery` | Task enqueueing |

## Notes
- Generate `request_id` with `uuid.uuid4()`.
- Store initial status in Redis immediately on enqueue.
- Router registered in `src/main.py` with prefix `/`.
