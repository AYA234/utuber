# Task 06 — Cloud Storage Integration (Large-File Fallback)

**Priority:** P0  
**Estimate:** 2 h  
**Assignee:** sw_developer  
**Depends on:** Task 01  
**Plan refs:** PLAN.md §2.5 (email delivery + size fallback), §8.3 (emailer spec), §9 (env vars), backlog #8

## Description
Implement a cloud storage helper used by the email service when the approved 480p video exceeds `MAX_ATTACHMENT_MB`. Upload the file and return a time-limited download URL.

## Acceptance Criteria
- `upload_temp(file_path) → str` uploads the file and returns a pre-signed/expiring URL.
- URL expiry controlled by `CLOUD_STORAGE_LINK_TTL_HOURS` env var (default `48`).
- Bucket name from `CLOUD_STORAGE_BUCKET` env var.
- Raises `CloudStorageError` on upload failure.
- Uploaded object is set to auto-delete after TTL.

## Supported Backend
Start with **Google Cloud Storage** (matches Gemini/GCP stack). Abstract behind a thin interface so the backend can be swapped.

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `google-cloud-storage` | GCS client |

## Env Vars
- `CLOUD_STORAGE_BUCKET` — required for V1.
- `CLOUD_STORAGE_LINK_TTL_HOURS` — default `48`.
- `GOOGLE_APPLICATION_CREDENTIALS` — path to service account JSON (or Workload Identity).

## Notes
- This is **required for V1** — the email service depends on it for the size fallback (PLAN.md §2.5, §9).
- The 480p file is deleted from local temp after a successful upload or email send.
