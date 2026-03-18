# Task 11 — Unit Tests

**Priority:** P1  
**Estimate:** 3–4 h  
**Assignee:** qa_tester  
**Depends on:** Tasks 02–08  
**Plan refs:** PLAN.md §6 (test files), backlog #11

## Description
Write unit tests for all core services, using mocks for external dependencies (Gemini API, yt-dlp, SMTP, Redis).

## Acceptance Criteria
- Coverage ≥ 80% across `src/`.
- All tests pass with `pytest`.
- No real network calls in unit tests (all external I/O mocked).

## Test Files & Coverage

| File | What to test |
|---|---|
| `tests/test_api.py` | Valid/invalid POST /request, GET /status, 404 on unknown id |
| `tests/test_analyzer.py` | Policy prompt format, Gemini response parsing, malformed response error, file cleanup on exception |
| `tests/test_downloader.py` | URL validation, mode selection, error cases (private video, duration exceeded) |
| `tests/test_emailer.py` | Attachment path (≤ limit), cloud link path (> limit), rejection email (no category leakage) |

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `pytest` | Test runner |
| `pytest-asyncio` | Async test support |
| `httpx` | FastAPI test client |
| `pytest-mock` / `unittest.mock` | Mocking external services |

## Notes
- Use `pytest-cov` to measure coverage: `pytest --cov=src --cov-report=term-missing`.
- Parameterize URL validation tests (valid + invalid URL list).
