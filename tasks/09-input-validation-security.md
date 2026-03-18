# Task 09 — Input Validation & Security

**Priority:** P1  
**Estimate:** 1–2 h  
**Assignee:** sw_developer  
**Depends on:** Task 02  
**Plan refs:** PLAN.md §11 (security considerations), backlog #10

## Description
Harden the API layer with strict input validation and basic security measures.

## Acceptance Criteria
- YouTube URL: allow only `youtube.com/watch?v=` and `youtu.be/` patterns (whitelist). Reject everything else with `422`.
- Email: RFC-compliant format validation via Pydantic `EmailStr`.
- All validation errors return structured JSON with clear messages.
- API keys and secrets never appear in logs or responses.
- All dependencies pinned with exact versions in `requirements.txt`.

## Checks to Implement
| Check | Location |
|---|---|
| YouTube URL whitelist | `src/models.py` (Pydantic validator) |
| Email format | `src/models.py` (`EmailStr`) |
| Max URL length (2048 chars) | `src/models.py` |
| Secrets not logged | `src/config.py` (`model_config = {"hide_input_in_errors": True}`) |

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `pydantic[email]` | `EmailStr` validator |
| `python-multipart` | Form/body parsing (FastAPI dep) |

## Notes
- Do not expose internal error details (stack traces) in production responses.
- Rate limiting is a separate task (Task 10).
