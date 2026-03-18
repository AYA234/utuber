# Task 05 — Gemini Analyzer Service

**Priority:** P0  
**Estimate:** 3 h  
**Assignee:** sw_developer  
**Depends on:** Task 03, Task 04  
**Plan refs:** PLAN.md §2.2 (AI model), §8.1 (analyzer spec), §5 (verdict schema), backlog #3

## Description
Implement `src/services/analyzer.py` that uploads a 144p video file to the Gemini Files API and returns a structured moderation verdict.

## Acceptance Criteria
- `analyze(file_path, policy)` returns `{approved, violated_categories, reasoning, confidence}`.
- Uses Gemini 2.0 Flash (`gemini-2.0-flash`) by default; configurable via `GEMINI_MODEL` env var.
- Uploads the full video file via the Gemini Files API (not base64 inline).
- Deletes the uploaded file from Gemini after receiving the verdict.
- Parses structured JSON response; raises `ModerationParseError` if response is malformed.
- Raises `GeminiAPIError` on API failure; caller should retry.

## Prompt Contract
Inject the formatted policy string from `policy.py` into the prompt. Instruct Gemini to:
1. Watch the full video.
2. Evaluate each listed policy category.
3. Return a JSON object matching the verdict schema (PLAN.md §5).

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `google-genai` | Gemini API client |
| `google-auth` | Authentication |

## Env Vars
- `GEMINI_API_KEY` — required.
- `GEMINI_MODEL` — default `gemini-2.0-flash`.

## Notes
- The 144p file gives full visual + audio coverage with no missed windows (PLAN.md §2.2, §2.3).
- Always delete the Gemini file upload even if analysis raises an exception (use `finally`).
