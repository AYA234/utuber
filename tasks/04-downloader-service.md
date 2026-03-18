# Task 04 — Downloader Service

**Priority:** P0  
**Estimate:** 2 h  
**Assignee:** sw_developer  
**Depends on:** Task 01  
**Plan refs:** PLAN.md §2.3 (download strategy), §8.2 (downloader spec), backlog #5

## Description
Implement `src/services/downloader.py` using `yt-dlp` with two download modes:

| Mode | Quality | Purpose |
|---|---|---|
| `moderation` | 144p (lowest available) | Full video for AI analysis |
| `delivery` | 480p (`DOWNLOAD_QUALITY`) | Approved video for emailing |

## Acceptance Criteria
- `download(url, mode)` returns `{file_path, title, duration_seconds, file_size_bytes}`.
- Moderation download: ≤ 20 MB for typical videos, full video (no trimming).
- Delivery download: respects `DOWNLOAD_QUALITY` env var (default `480`).
- Files saved to `TEMP_DOWNLOAD_DIR` with unique filenames.
- Raises `VideoUnavailableError` for private/deleted videos.
- Raises `VideoDurationError` if duration > `MAX_VIDEO_DURATION_MINUTES`.
- Files are **not** auto-deleted by this service (caller is responsible).

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `yt-dlp` | YouTube download |

## Notes
- Use `yt-dlp`'s `format` selector: `bestvideo[height<=144]+bestaudio/best[height<=144]` for moderation mode.
- Do not import `subprocess` directly — use `yt_dlp.YoutubeDL` Python API.
- Validate that URL domain is `youtube.com` or `youtu.be` before downloading (security — PLAN.md §11).
