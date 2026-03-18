# Task 13 — Integration Tests

**Priority:** P2  
**Estimate:** 3 h  
**Assignee:** qa_tester  
**Depends on:** Task 11, Task 12  
**Plan refs:** PLAN.md backlog #12

## Description
End-to-end tests that exercise the real Gemini API and a live Redis instance. Run in CI only (not on every local `pytest` run).

## Acceptance Criteria
- `pytest -m integration` runs the integration suite.
- At minimum, two test scenarios:
  1. **Approved flow**: submit a known-clean YouTube URL → poll until `completed` → verify an email would be sent (mock SMTP, real Gemini + yt-dlp).
  2. **Rejected flow**: submit a URL with known policy violation → poll until `rejected` → verify rejection email would be sent.
- Tests skip automatically if `GEMINI_API_KEY` is not set (CI gate).
- Redis started via `docker-compose` before the suite runs.

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `pytest` + `pytest-asyncio` | Test runner |
| `httpx` | HTTP client for API calls |
| `pytest-mark` | `@pytest.mark.integration` marker |
| Real Redis | Via Docker Compose |
| Real Gemini API | `GEMINI_API_KEY` required |

## Notes
- Keep integration tests in `tests/integration/` to separate from unit tests.
- Use `pytest.ini` or `pyproject.toml` to register the `integration` marker.
- Do not commit real video files; use short, public YouTube videos (< 2 min) as fixtures.
