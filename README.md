# SixthSense

> **See the risk. Stop it early.** Early, private, verifiable action for the safety of women and
> children, without ever becoming a tool to track the people it protects.

SixthSense helps people **avoid** risky routes, **reduce** risk during a journey, **interrupt**
harm fast, and **fix** the places that cause it.

| | |
|---|---|
| **Deploy to a server** | [`DEPLOY.md`](DEPLOY.md): one command on Ubuntu 24.04 |
| **Design** | [`Docs/`](Docs/) (the product is called *SafeThread* in the design documents) |
| **API contract** | [`contracts/openapi.yaml`](contracts/openapi.yaml) |
| **Platform** | Android app (Flutter); iOS later |

## Features

**For the person travelling (no ID ever needed)**
- Email sign-in with a device-bound key; the phone stays signed in.
- Journeys with check-in timers, a real PIN and a duress PIN. Timers run on the phone and
  survive the app being closed; an open journey is resumed when the app reopens.
- SOS: button, volume buttons, voice words, quick-settings tile and home-screen widget; covert
  SOS; offline SMS fallback.
- Trusted contacts get alerts in steps (push, Telegram, email) until someone answers.
- End-to-end encrypted live location and live audio, only to the people helping right now.
- Ranked walking routes using open data and anonymous place reports.
- Walk With Me: ask a nearby verified companion to walk with you, with a handshake code.
- Rate the route, the companion and every responder who came.

**For helpers (verified)**
- Identity verification: ID photo + selfie face match (self-hosted CompreFace), ID-photo OCR on
  the phone, optional encrypted face reference.
- A fresh face scan before a companion or responder sees anyone's location.
- Nearby responders are asked during a full alert, closest first.

**Safety and moderation**
- A 1-star rating suspends a companion or responder from helping straight away.
- Admin page (`/admin/feedback`): every rating, CSV downloads, and a queue to **Ban** or
  **Restore**. A ban blocks the account and keeps its ID hashes and face embedding so the
  person can't verify again. Admins never see who a helper is.

**Privacy by design**
- No live location stored, no ID documents or ID numbers stored (keyed hashes only), no route
  kept after a walk. Anonymous place reports live in a separate service with its own database.
- Every sensitive action is written to an append-only audit log without personal data.

## Repository

| Path | What | Stack |
|---|---|---|
| `mobile/` | Android app | Flutter |
| `backend/` | Core API: accounts, journeys, alerts, helpers, verification, feedback | Python 3.12, FastAPI, SQLAlchemy, Alembic, uv |
| `insight/` | Routes and anonymous place reports (own database, no user ids) | Same stack |
| `dashboard/` | Campus map dashboard | Next.js + MapLibre |
| `infra/` | Postgres (x2), Redis, NATS, Mailpit, CompreFace | Docker Compose |
| `deploy/` | Production installer, HTTPS, services, `.env` generator | Bash, systemd, Caddy |
| `routing-engine/` | Optional self-hosted Valhalla | Docker |

## Run locally (development)

Needs Docker, [uv](https://docs.astral.sh/uv/), Node 22 and Flutter.

```bash
cp .env.example .env
docker compose -f infra/docker-compose.yml up -d --wait
(cd backend && uv sync && uv run alembic upgrade head)
(cd insight && uv sync && uv run alembic upgrade head)

cd backend && uv run uvicorn app.main:app --reload --port 8000   # Core API  -> /docs
cd backend && uv run arq app.worker.WorkerSettings                # alert worker
cd insight && uv run uvicorn app.main:app --reload --port 8001   # Insight API
cd mobile  && flutter run                                         # emulator or USB phone
```

Sign-in codes land in the local Mailpit inbox at http://localhost:8025. Without CompreFace on
your machine, set `FACE_MATCHER=mock` in `.env` (dev only).

## Checks

```bash
(cd backend && uv run ruff check . && uv run pytest -q)
(cd insight && uv run ruff check . && uv run pytest -q)
(cd mobile  && flutter analyze && flutter test)
(cd dashboard && corepack pnpm install && corepack pnpm lint && corepack pnpm build)
```
