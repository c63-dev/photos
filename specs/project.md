# Project Context

## Purpose
C63 Photos is a single-user, self-hosted photo and video library. It exposes a directory tree as a browsable gallery and file manager, and adds a metadata index for timeline, search, albums, duplicates and map browsing. The server validates paths, lists directories, generates and caches preview images, serves and downloads files, performs file-management operations, and delivers the application shell and its configuration to the browser.

These specs were generated retrospectively from current server behaviour and describe what the system does today.

## Environment
- Optional server capabilities extend behaviour when present: an image resizer, a converter for non-browser image formats, a video frame extractor, a PDF renderer, and archive (zip) support. Absence of a capability degrades gracefully rather than failing.
- Frontend assets (scripts, styles, language packs) load from a public CDN by default or from a configured self-hosted location.

## Suggested tooling (non-binding)
The capability specs never name tools. For implementers, the suggested providers for the media capabilities are:
- libheif for decoding HEIC/HEIF images (thumbnails, on-demand conversion and background transcoding to JPEG).
- FFmpeg for video: frame extraction for thumbnails, HEVC decoding, transcoding to H.264 in MP4, and stream probing to detect HEVC regardless of container.

Any tool that satisfies the behaviour in the specs is acceptable. Nothing is installed or assumed present without explicit permission.

## Conventions
- Every server entry point is a request with an `action` parameter. A request with no action returns the main document.
- Paths exchanged with the browser are always relative to the configured root, never absolute.
- Any item whose name starts with `_files` is reserved and hidden from listings.
- Storage (configuration, cache, users, custom includes) lives under a configurable storage directory, by default `_files` next to the application.
- Error responses are an HTML page with an HTTP status for GET requests and a JSON object with an `error` message for POST requests.
- Every response carries a diagnostic header listing what happened, elapsed time and peak memory.

## Scope of these specs
- In scope: server-side behaviour, including every request the frontend UI can trigger.
- Out of scope: implementation details of any kind, frontend rendering and interaction, licensing and commercial features, third-party gallery integration, the optional tasks plugin.
