# Task 07 — Email Service & HTML Templates

**Priority:** P0  
**Estimate:** 3 h  
**Assignee:** sw_developer  
**Depends on:** Task 01, Task 06  
**Plan refs:** PLAN.md §2.5 (delivery contract), §8.3 (emailer spec), §9 (env vars), backlog #6, #9

## Description
Implement `src/services/emailer.py` and the two Jinja2 HTML templates.

### Two email paths
| Path | Trigger | Content |
|---|---|---|
| Approved | `verdict.approved == True` | Video title, thumbnail. Attach 480p file **or** expiring cloud link if file > `MAX_ATTACHMENT_MB`. |
| Rejected | `verdict.approved == False` | Generic "content not available" message. **Never** expose `violated_categories` to the user. |

## Acceptance Criteria
- `send_approved(email, video_meta, file_path, verdict)` — attaches file if ≤ `MAX_ATTACHMENT_MB`; uploads to cloud and sends link otherwise.
- `send_rejected(email, video_meta)` — sends generic rejection; no policy details.
- Both use Jinja2 templates from `src/templates/`.
- SMTP backend: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`.
- SendGrid backend: `SENDGRID_API_KEY` (activated when key is present).
- 480p temp file deleted after send.

## Templates (`src/templates/`)
- `approved.html` — responsive HTML email: video title, optional thumbnail, CTA button.
- `rejected.html` — responsive HTML email: polite generic message.

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `Jinja2` | Email template rendering |
| `sendgrid` | SendGrid backend (optional) |
| `smtplib` + `email.mime` | SMTP backend (stdlib) |

## Env Vars
See PLAN.md §9 for full list (`SMTP_*`, `SENDGRID_API_KEY`, `MAX_ATTACHMENT_MB`, `EMAIL_FROM`).

## Notes
- Backend selected automatically: use SendGrid if `SENDGRID_API_KEY` is set, otherwise SMTP.
- Keep violation categories server-side only (PLAN.md §11).
