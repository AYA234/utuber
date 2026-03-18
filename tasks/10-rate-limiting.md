# Task 10 — Rate Limiting & Abuse Prevention

**Priority:** P2  
**Estimate:** 2 h  
**Assignee:** sw_developer  
**Depends on:** Task 02  
**Plan refs:** PLAN.md §11 (rate limiting), backlog #14

## Description
Add per-IP and per-email rate limiting to the `POST /request` endpoint, and detect duplicate requests.

## Acceptance Criteria
- Per-IP: max 10 requests / hour. Exceeding returns `429` with `Retry-After` header.
- Per-email: max 5 requests / hour per email address.
- Duplicate detection: if the same `(youtube_url, email)` pair has a non-failed, non-rejected request within the last 10 minutes, return the existing `request_id` instead of creating a new task.
- Counters stored in Redis with TTL.
- Limits configurable via env vars (`RATE_LIMIT_PER_IP`, `RATE_LIMIT_PER_EMAIL`, `DEDUP_WINDOW_MINUTES`).

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `slowapi` | FastAPI rate limiting (wraps `limits`) |
| `redis` | Counter + dedup store |

## Notes
- Use `X-Forwarded-For` header for IP detection behind a proxy (trust only first IP).
- Hash `(url, email)` for dedup key to avoid storing raw email in Redis keys.
