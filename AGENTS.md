# Neon Rave — Base44 Setup

## Project overview
Single-file static HTML game ("Neon Rave") with inline CSS/JS and one MP3 audio track. No build step, no backend, no package manager.

## Running in the sandbox
- `docker compose -f docker-compose.base44.yml up -d` serves the repo root via `nginx:alpine` on host port 3000.
- The source (`index.html`, `*.mp3`) is bind-mounted read-only — edits appear on browser refresh; no rebuild needed.
- No external secrets or environment variables required.

## Verifying
- `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/` should return `200`.
- The start screen should show "NEON RAVE" with a Start button.
