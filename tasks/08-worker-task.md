# Task 08 — Celery Worker Task (Full Pipeline)

**Priority:** P0  
**Estimate:** 2 h  
**Assignee:** sw_developer  
**Depends on:** Task 02, Task 04, Task 05, Task 07  
**Plan refs:** PLAN.md §3 (system flow), §8.4 (worker spec), backlog #7

## Description
Implement the single Celery task `process_video_request` in `src/workers/tasks.py` that orchestrates the full moderation + delivery pipeline.

## Pipeline Steps
```
1. Set status → "analyzing"
2. Download 144p (moderation copy)
3. Analyze with Gemini → verdict
4. Delete 144p temp file
5a. APPROVED → set status "downloading" → download 480p → set status "sending" → email with attachment/link → set status "completed"
5b. REJECTED  → set status "sending" → send rejection email → set status "rejected"
6. On any error → set status "failed", log error
```

## Acceptance Criteria
- Redis status updated at every step (step names above).
- Retries on transient errors (network, Gemini rate limit): max 3 retries, exponential backoff (2s, 4s, 8s).
- Hard timeout: 10 minutes (`CELERY_TASK_SOFT_TIME_LIMIT=540`, `CELERY_TASK_TIME_LIMIT=600`).
- Temp files cleaned up in all code paths (success, rejection, and error).
- Task signature: `process_video_request(request_id: str, youtube_url: str, email: str)`.

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `celery[redis]` | Task queue |
| `redis` | Status store |

## Notes
- Use `self.retry(exc=..., countdown=2**self.request.retries)` pattern.
- Log `request_id` in every log line for traceability.
