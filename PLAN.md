# UTuber — Architecture & Implementation Plan

## 1. Overview

A service that receives a YouTube video link and an email address, sends the video to an AI model for content moderation against the **Netfree policy** (content guidelines for ultra-orthodox communities), and either delivers the approved video via email or sends a rejection notice.

---

## 2. Key Design Decisions

### 2.1 Language: **Python 3.12+**
- Best ecosystem for AI integration, video processing, and web APIs.
- Libraries: FastAPI, Celery, yt-dlp, google-genai, Jinja2.

### 2.2 AI Model: **Google Gemini 2.0 Flash**
- Fast, cost-effective, and suitable for multimodal moderation tasks.
- Structured output support (JSON verdict).
- Alternately Gemini 2.5 Flash can be used for higher accuracy at slightly higher cost.
- The service downloads the **full video at minimal quality (144p)** for moderation. This gives Gemini 100% visual and audio coverage with a small file, eliminating the false-approval risk of sampled frames or clips.

### 2.3 Download Strategy
| Step | Action |
|---|---|
| **Moderation** | Download the **full video at 144p** via `yt-dlp`. Small file (~5–20 MB), full coverage — no missed content windows. Sent to Gemini for policy evaluation. |
| **Approved** | Download the video again at **delivery quality (480p)** for emailing. |
| **Rejected** | The 144p moderation copy is deleted. No delivery download performed. |

Two-pass download approach: the 144p moderation copy is cheap and fast (~10–30s), guarantees complete coverage, and is discarded after the verdict. The delivery download only runs for approved videos.

### 2.4 Service Architecture: **FastAPI + Celery + Redis**
- **FastAPI** — Lightweight async REST API to receive requests.
- **Celery** — Background task queue for processing (video analysis is slow, 30s–3min).
- **Redis** — Message broker for Celery + status/state store.
- The user gets an immediate response with a request ID, and can poll for status.

### 2.5 Email Delivery
- **SMTP** (Gmail/Outlook) for simple setups, or **SendGrid/Mailgun** for production.
- **Default delivery: email attachment.** The 480p download is attached directly to the approval email.
- **Size fallback**: if the file exceeds the configured limit (default 20 MB), the service uploads it to temporary cloud storage and sends a time-limited download link instead. This fallback is part of V1.
- `MAX_ATTACHMENT_MB` controls the threshold; the cloud storage bucket is required even for V1.

---

## 3. System Flow

```
User Request (URL + Email)
        │
        ▼
   ┌─────────┐
   │ FastAPI  │──→ Validate URL & email ──→ Enqueue task ──→ Return request_id
   └─────────┘
        │
        ▼ (async via Celery)
   ┌──────────────────┐
   │  1. Validate URL  │ ── Is it a real YouTube link?
   └────────┬─────────┘
            ▼
   ┌──────────────────────────┐
   │  2. Download 144p copy   │ ── Full video, minimal size (~5-20 MB)
   │     → send to Gemini     │ ── With Netfree policy prompt
   └────────┬─────────────────┘
            ▼
       ┌─────────┐
       │Approved?│
       └────┬────┘
        YES │         NO
            ▼          ▼
   ┌────────────┐  ┌──────────────────┐
   │ 3. Download │  │ Send rejection   │
   │  via yt-dlp │  │ email with reason │
   └─────┬──────┘  └──────────────────┘
         ▼
   ┌─────────────────────────────────────┐
   │ 4. Email 480p file as attachment     │
   │    (or cloud link if file > 20 MB)   │
   └─────────────────────────────────────┘
```

---

## 4. Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| Language | Python 3.12+ | Core language |
| Web Framework | FastAPI | REST API, request handling |
| Task Queue | Celery | Async background processing |
| Message Broker | Redis | Celery broker + status store |
| AI Model | Google Gemini 2.0 Flash | Video content analysis |
| Video Download | yt-dlp | YouTube video downloading |
| Email | smtplib + email.mime / SendGrid | Email delivery |
| Email Templates | Jinja2 | HTML email templates |
| Config | pydantic-settings + YAML | App config + policy rules |
| Containerization | Docker + Docker Compose | Deployment |
| Testing | pytest + pytest-asyncio | Unit & integration tests |

---

## 5. Netfree Policy — Content Moderation Prompt

The policy will be loaded from a YAML configuration file and injected into the Gemini prompt. Initial categories to check:

```yaml
# config/netfree_policy.yaml
policy:
  name: "Netfree Content Policy"
  version: "1.0"
  description: "Content moderation for ultra-orthodox community"
  
  # Categories to flag — video is REJECTED if any category is violated
  categories:
    - id: immodesty
      description: "Immodest dress or appearance (women not dressed modestly, exposed arms/legs/neckline)"
      severity: block
    - id: romantic_content
      description: "Romantic or intimate scenes, dating content, affectionate physical contact between genders"
      severity: block
    - id: profanity
      description: "Vulgar language, cursing, blasphemy"
      severity: block
    - id: violence
      description: "Graphic violence, gore, disturbing imagery"
      severity: block
    - id: secular_ideology
      description: "Content promoting ideologies contrary to Torah values"
      severity: block
    - id: inappropriate_music
      description: "Music with inappropriate lyrics or female singing (kol isha)"
      severity: block
    - id: mixed_swimming
      description: "Mixed-gender swimming or beach scenes"
      severity: block

  # Verdict format expected from the model
  verdict_schema:
    approved: boolean
    violated_categories: list
    reasoning: string
    confidence: float
```

> **Note**: Full policy settings will be provided later and this file will be updated.

---

## 6. Project Structure

```
utuber/
├── PLAN.md                       # This file
├── README.md                     # Setup & usage instructions
├── requirements.txt              # Python dependencies
├── .env.example                  # Environment variable template
├── docker-compose.yml            # Redis + API + Worker
├── Dockerfile                    # App container
│
├── config/
│   └── netfree_policy.yaml       # Moderation policy rules
│
├── src/
│   ├── __init__.py
│   ├── main.py                   # FastAPI app entrypoint
│   ├── config.py                 # Settings (pydantic-settings)
│   ├── models.py                 # Pydantic request/response models
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes.py             # POST /request, GET /status/{id}
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── analyzer.py           # Gemini video analysis + policy check
│   │   ├── downloader.py         # yt-dlp video download
│   │   ├── emailer.py            # Email composition & sending
│   │   └── policy.py             # Load & format Netfree policy
│   │
│   ├── workers/
│   │   ├── __init__.py
│   │   └── tasks.py              # Celery task: process_video_request
│   │
│   └── templates/
│       ├── approved.html          # "Your video is ready" email
│       └── rejected.html          # "Video not available" email
│
└── tests/
    ├── __init__.py
    ├── test_api.py
    ├── test_analyzer.py
    ├── test_downloader.py
    └── test_emailer.py
```

---

## 7. API Contract

### `POST /request`
Submit a new video moderation + delivery request.

**Request Body:**
```json
{
  "youtube_url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
  "email": "user@example.com"
}
```

**Response (202 Accepted):**
```json
{
  "request_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "queued",
  "message": "Your request is being processed. You will receive an email shortly."
}
```

### `GET /status/{request_id}`
Check the status of a request.

**Response (200 OK):**
```json
{
  "request_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "analyzing | downloading | sending | completed | rejected | failed",
  "detail": "Video is being analyzed for content policy compliance.",
  "created_at": "2026-03-18T10:30:00Z"
}
```

---

## 8. Component Details

### 8.1 Analyzer Service (`analyzer.py`)
- Loads the Netfree policy from YAML.
- Constructs a structured prompt instructing Gemini to evaluate each policy category against the full video.
- Uploads the 144p moderation copy to the Gemini Files API and passes the file URI in the prompt — full video, zero missed windows.
- Deletes the uploaded file from Gemini after the verdict is received.
- Parses the structured JSON response: `{approved, violated_categories, reasoning, confidence}`.
- Returns the verdict to the worker task.

### 8.2 Downloader Service (`downloader.py`)
- Uses `yt-dlp` to download the video.
- Produces two modes:
  - **Moderation mode**: download the full video at `144p` (or the lowest available quality). Used for AI policy evaluation — complete coverage, small file.
  - **Delivery mode**: download the approved video at `480p` (configurable via `DOWNLOAD_QUALITY`) for emailing.
- Both downloads use a temp directory; files are deleted after use.
- Returns the file path and metadata (title, duration, file size).

### 8.3 Email Service (`emailer.py`)
- Two email paths:
  - **Approved**: HTML email with video title and thumbnail. The 480p file is **attached directly** to the email. If the file exceeds `MAX_ATTACHMENT_MB`, it is uploaded to cloud storage and the email contains a time-limited download link instead.
  - **Rejected**: HTML email explaining the video didn't pass content policy (generic message — specific violated categories are logged server-side only, never exposed to the user).
- Uses Jinja2 templates for email body.
- Supports SMTP and SendGrid backends (configurable).

### 8.4 Worker Task (`tasks.py`)
- Single Celery task: `process_video_request(request_id, youtube_url, email)`
- Orchestrates the full flow: validate → analyze → download/reject → email.
- Updates status in Redis at each step.
- Retries on transient failures (network, API rate limits) with exponential backoff.
- Timeout: 10 minutes max per request.

---

## 9. Configuration & Environment

```bash
# .env.example
GEMINI_API_KEY=your-gemini-api-key
REDIS_URL=redis://localhost:6379/0

# Email - SMTP
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
EMAIL_FROM=UTuber <noreply@utuber.app>

# Email - SendGrid (alternative)
# SENDGRID_API_KEY=your-sendgrid-key

# App
MAX_VIDEO_DURATION_MINUTES=60
DOWNLOAD_QUALITY=480
MODERATION_QUALITY=144
MAX_ATTACHMENT_MB=20
TEMP_DOWNLOAD_DIR=/tmp/utuber

# Cloud storage (required for large-file fallback in V1)
# CLOUD_STORAGE_BUCKET=your-bucket-name
# CLOUD_STORAGE_LINK_TTL_HOURS=48
```

---

## 10. Initial Backlog

| # | Task | Priority | Assignee | Acceptance Criteria |
|---|---|---|---|---|
| 1 | Project scaffolding & Docker setup | P0 | sw_developer | FastAPI app starts, Redis connects, Celery worker runs |
| 2 | API endpoints (POST /request, GET /status) | P0 | sw_developer | Endpoints accept valid input, return proper responses, reject invalid input |
| 3 | Gemini analyzer service | P0 | sw_developer | Sends moderation artifacts to Gemini with policy prompt, parses structured verdict |
| 4 | Netfree policy loader | P1 | sw_developer | Loads YAML config, formats into prompt, supports category toggling |
| 5 | yt-dlp downloader service (two modes) | P0 | sw_developer | Downloads 144p for moderation and 480p for delivery; both modes return file path and metadata |
| 6 | Email service (approved + rejected) | P0 | sw_developer | Attaches file when ≤ MAX_ATTACHMENT_MB; uploads to cloud and sends link when larger |
| 7 | Celery worker task (full pipeline) | P0 | sw_developer | Orchestrates full flow, updates status, handles retries |
| 8 | Cloud storage integration (large-file fallback) | P0 | sw_developer | Files exceeding MAX_ATTACHMENT_MB uploaded to cloud bucket; email contains expiring download link |
| 9 | HTML email templates | P1 | sw_developer | Clean, responsive email templates for both outcomes, supporting both attachment and link variants |
| 10 | Input validation & security | P1 | sw_developer | URL validation (YouTube only), email validation, rate limiting |
| 11 | Unit tests | P1 | qa_tester | Core services tested with mocks, >80% coverage |
| 12 | Integration tests | P2 | qa_tester | End-to-end flow tested with real Gemini API |
| 13 | Docker Compose production config | P2 | sw_developer | Multi-container setup, health checks, volume mounts |
| 14 | Rate limiting & abuse prevention | P2 | sw_developer | Per-email rate limit, duplicate request detection |

---

## 11. Security Considerations

- **Input validation**: Strict YouTube URL pattern matching (whitelist `youtube.com` and `youtu.be` domains only).
- **Email validation**: Proper format validation, no open relay.
- **Rate limiting**: Per-IP and per-email limits to prevent abuse.
- **API key protection**: All secrets in environment variables, never in code.
- **Temp file cleanup**: Download files deleted immediately after email sent.
- **No policy details in rejection emails**: Generic message to users — specific violation categories are logged server-side only.
- **Dependency pinning**: All dependencies pinned in `requirements.txt`.

---

## 12. Future Enhancements (Out of Scope for V1)

- Web UI for submitting requests and tracking status.
- Webhook/callback support instead of polling.
- Audio-only extraction option.
- Batch processing (multiple videos per request).
- Admin dashboard for monitoring and policy tuning.
- Caching: skip re-analysis for previously approved videos.
- Multi-language policy support.
