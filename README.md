# Cirql

**Multi-Tenant WhatsApp Engagement Platform**

Cirql is a platform built and operated by [Lotus Webtek](https://lotuswebtek.com) for organizations that run structured recurring-session programs -- tutoring, mentoring, coaching, community meetups, and similar formats.

It automates session reminders (text and voice), attendance tracking, participant check-ins, and coordinator reporting entirely through WhatsApp. Participants never install anything or learn a new tool.

---

## What It Does

- Sends session reminders via WhatsApp -- text or voice, in the participant's preferred language
- Tracks confirmations, cancellations, and no-shows automatically
- Runs post-session check-ins and learner pulse checks via button taps
- Alerts coordinators to patterns -- missed sessions, mismatches, confidence drops
- Supports multiple engagement formats: 1:1 pairs, small groups, open events, cohort sessions
- Fully multi-tenant -- one codebase, each client configured independently with no code changes

---

## Reference Implementation

**OneProsper** is the first tenant. OneProsper runs a structured buddy-mentoring program pairing North American volunteer buddies with learners in India for weekly Sunday sessions.

Every behavior OneProsper requires -- Sunday-only scheduling, 1:1 pairs, bilingual Hindi/English messages, 10-session programs -- is expressed as tenant configuration, not hardcoded logic.

---

## Architecture

Cirql runs as a single deployable application on AWS Lightsail using Docker Compose.

| Component | Technology |
|---|---|
| API server | Node.js / Express |
| Scheduler | pg_cron (PostgreSQL) |
| Job queue | Redis + Bull (voice generation) |
| Database | PostgreSQL |
| Frontend | React |
| Voice TTS | ElevenLabs |
| WhatsApp | Meta Cloud API (direct) |
| Reverse proxy | nginx |

Configuration cascades five levels: System default -> Tenant -> Program -> Engagement -> Participant. The most specific non-null value wins at each level.

---

## WABA Model

Cirql uses a hybrid WhatsApp Business Account model:

- **Shared WABA** (Lotus Webtek-owned) -- Starter-tier tenants share one platform number. Zero Meta onboarding required. New tenants go live in minutes.
- **Tenant-owned WABA** -- Pro and Enterprise tenants connect their own WABA via Meta Embedded Signup. Messages show the tenant's own brand identity and quality rating is isolated.

---

## Key Documents

| Document | Description |
|---|---|
| [`cirql-prd.md`](./cirql-prd.md) | Full Product Requirements Document -- all phases, data model, config cascade, voice reminders, engagement types |
| [`cirql-architecture.md`](./cirql-architecture.md) | System Architecture and Task Register -- deployment stack, schema, phase task checklists |

---

## Design Session

The full product design session that produced this architecture is documented here:

[Cirql Design Session -- Claude.ai](https://claude.ai/share/efe1e9e6-0c72-42be-88cf-19acf51b2f4c)

This session covers the evolution from a single-tenant OneProsper system to the Cirql multi-tenant product, including decisions on WABA ownership, engagement type generalization, configuration cascade design, and voice reminder architecture.

---

## Development Setup

```bash
# Clone the repo
git clone https://github.com/lotuswebtek/cirql.git
cd cirql

# Copy environment file
cp .env.example .env
# Fill in required values in .env

# Start all services
docker compose up -d

# Run database migrations
docker compose exec cirql-api npm run migrate
```

---

## Environment Variables

See `.env.example` for the full list. Key variables:

| Variable | Description |
|---|---|
| `PORT` | API server port |
| `DB_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `WA_TOKEN` | Meta WhatsApp Cloud API token |
| `WA_PHONE_ID` | Meta phone number ID |
| `WA_WEBHOOK_SECRET` | Webhook verification secret |
| `ELEVENLABS_API_KEY` | ElevenLabs TTS API key |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID |
| `JWT_SECRET` | JWT signing secret |
| `ENCRYPTION_KEY` | pgcrypto key for phone number encryption |

---

## License

Proprietary. Built and maintained by Lotus Webtek.

---

*Cirql -- Lotus Webtek -- SP-2025-002*
