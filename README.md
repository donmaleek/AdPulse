# AdPulse Africa (AAIS) — Autonomous Ad Intelligence System
**Run ads like a pro. No agency needed.**  
**Print attention on demand → convert it into deposits, bookings, and repeat customers.**

AdPulse Africa is a production-grade, OSS-first platform for African businesses (real estate, clinics, coaches, e-commerce, events, hospitality) that:
- **launches and tests ads automatically (A/B/C)**
- **tracks conversions server-side (truth layer)**
- **moves leads through WhatsApp + booking + deposit enforcement**
- **rebalances budgets with guardrails**
- **drives reviews + Google Business Profile optimization**
- **centralizes data into one dashboard**

> Core loop: **Attention Engine → Lead Engine → Booking/Payment Engine → Retention Engine**  
All measured end-to-end with profit-first metrics.

---

## Table of Contents
- [Key Capabilities](#key-capabilities)
- [Architecture](#architecture)
- [Monorepo Layout](#monorepo-layout)
- [Tech Stack](#tech-stack)
- [Core Concepts](#core-concepts)
- [Decision Engine Guardrails](#decision-engine-guardrails)
- [Metrics, SLAs, and Cost Ceilings](#metrics-slas-and-cost-ceilings)
- [Quick Start (Local Dev)](#quick-start-local-dev)
- [Production Deployment](#production-deployment)
- [Environment Variables](#environment-variables)
- [Webhook Integrations](#webhook-integrations)
- [Database and Migrations](#database-and-migrations)
- [Security Model](#security-model)
- [Observability](#observability)
- [Testing](#testing)
- [Roadmap](#roadmap)

---

## Key Capabilities

### 1) AI Ad Orchestrator (Offer → Profit → Creatives → Test → Deploy → Optimize)
- Offer intake (product/service/event/property/clinic appointment/coaching)
- Contribution margin + **Max CPA** computation (profit intelligence)
- Generates **3 ad variations**:
  - A: direct offer
  - B: problem → solution
  - C: proof/authority
- Deploys to **Meta / Google / TikTok** (configurable)
- Monitors & optimizes:
  - pacing checks every **30–60 min**
  - budget rebalances every **6–12 hours**
  - creative fatigue scan **daily**
- Guardrails prevent runaway spend

### 2) WhatsApp + Booking Bot (Zero booking chaos)
- Inquiry capture + instant qualification
- Availability calendar integration
- Deposit link + auto verification
- Auto confirmations + reminders
- No-show follow-ups
- Broadcast campaigns + segmented winbacks

### 3) CRM + Lead Tracking (African follow-up reality)
- Standard pipeline stages
- Lead scoring (rules-first; model later)
- Assignment + follow-up automation
- Conversion visibility: lead → booking → paid deposit → served → review → repeat

### 4) M-Pesa + Payments (Deposit enforcement)
- STK push + Paybill reconciliation (provider-dependent)
- Paystack/Stripe optional for cards/international
- **Deposit required to confirm** with auto-expiry on unpaid holds
- Full audit log of all payment decisions

### 5) Reviews + Google Optimization
- Automated review requests post-service
- Negative review routing (private recovery flow)
- GBP cadence: photos/posts/Q&A updates with queue and tracking

### 6) Analytics Dashboard (Single source of truth)
- Executive metrics: **incremental profit**, MER, CAC/CPA, pipeline value
- Ads metrics: CTR/CPC/CPM/CPA, fatigue, learning status
- Ops metrics: booking conversion, no-show rate, deposit compliance, response speed
- Retention metrics: repeat rate, review velocity

---

## Architecture

**Traffic → Data Collection → Intelligence → Decision Engine → Execution**

### Execution Surfaces
- Meta Ads (FB/IG)
- Google Ads (Search/PMAX/YouTube)
- TikTok Ads
- WhatsApp, Email/SMS, Google Business Profile, SEO

### Data Truth Layer (non-negotiable)
- **Server-side events** written into Postgres first (truth)
- Optional forwarding to:
  - Meta Conversions API (CAPI)
  - Google Enhanced Conversions
- Web pixels are redundant (secondary) only

### Decision Engine
- Rules + Bayesian sequential tests (low-volume friendly)
- Spend caps, auto-pause, scale rules, anomaly detection
- All automated changes are recorded in **decision_logs** + **audit_logs**
- Rollback supported

---

## Monorepo Layout


adpulse-aaas/
apps/
web/ # Next.js dashboard + orchestrator UI
api/ # Fastify API gateway + webhooks
worker/ # BullMQ workers executing decision engine + automations
mobile/ # Expo (React Native) owner/sales companion app
packages/
shared/ # Zod schemas, enums, types (single source of truth)
connectors/ # Meta/Google/TikTok, WhatsApp, M-Pesa, Paystack, Stripe, GBP
decision-engine # pacing, tests, anomalies, scaling logic (pure functions)
ui/ # shared UI components
infra/
docker/ # compose.yaml + Caddyfile + scripts
observability/ # Prometheus/Grafana configs (optional)
database/
migrations/ # SQL migrations + materialized views
seeds/
docs/
runbooks/ # provider notes & operational docs


---

## Tech Stack

**Frontend**
- Next.js 14 + TypeScript
- Tailwind + shadcn/ui

**Backend**
- Node 20 + Fastify (API + webhooks)
- Redis + BullMQ (jobs/queues)
- Postgres (truth + CRM + booking + analytics)

**Optional OSS Add-ons**
- PostHog (product analytics)
- Metabase (BI)
- n8n (ops automation; non-money paths)

**Hosting**
- Docker Compose on a small VPS
- Caddy reverse proxy + automatic TLS

---

## Core Concepts

### Offer → Profit Intelligence
Each offer stores:
- price
- COGS / fulfillment costs
- payment fees
- refund allowance
- capacity constraints

Computed:
- **Contribution Margin** = price − COGS − delivery/ops − fees − refund allowance
- **Max CPA** = Contribution Margin × safety factor (default 0.60)

### A/B/C Experiment
Every launch includes:
- 3 variations with distinct angles
- minimum data thresholds before scale/kill actions
- bias for **fast learning** in low volume (Bayesian)

### Event Taxonomy (truth layer)
Minimum event types:
- `page_view`
- `lead_created`
- `whatsapp_started`
- `quote_sent`
- `booking_requested`
- `deposit_requested`
- `deposit_paid`
- `booking_confirmed`
- `service_delivered`
- `review_requested`
- `review_submitted`
- `repeat_purchase`

---

## Decision Engine Guardrails

Hard safety rules (defaults; configurable per org):
- Daily spend cap per campaign + per account
- Auto-pause if:
  - `CPA > 1.3 × MaxCPA` after `minClicks` or `minConversions`
- Budget increase approval required if:
  - increase > `X%` within 24 hours

Scaling rules:
- Only scale when:
  - `CPA <= MaxCPA` and `conversions >= threshold`
- Scale step:
  - +15% to +25% per 24h (configurable)
- Creative fatigue:
  - alert if CTR drops >20% AND CPA rises >15% (default)

All automated actions log:
- input metrics snapshot
- decision reason
- applied change
- rollback pointer

---

## Metrics, SLAs, and Cost Ceilings

### Latency Targets (p95)
- Webhook ingestion (M-Pesa/WA/Paystack): **< 500ms**
- Event write to DB: **< 150ms**
- Dashboard read queries: **< 2s** (materialized views + caching)

### Business KPIs
- Incremental Profit (profit after spend, fees, refunds, ops)
- CAC/CPA vs Max CPA
- MER (revenue / marketing spend)
- Lead-to-booking rate; booking-to-paid rate
- Time-to-first-response (WhatsApp + forms)
- Deposit compliance rate
- Repeat rate (30/60/90)
- Review rate + average rating

### Model/Decision Quality
- Forecast error (MAPE): ≤25% early, ≤15% mature
- Budget decision win-rate: ≥60% improve CPA within 48h
- Fatigue detection precision: ≥70%
- Lead score calibration: high-score converts ≥2× low-score

### Cost Ceilings
- Infra: target $20–$60/mo for SMB baseline (depends on volume)
- LLM usage: cap $0.03–$0.10 per lead equivalent
- Creative generation: template-first; paid generation only for winners

---

## Quick Start (Local Dev)

### Prereqs
- Node.js 20+
- pnpm 9+
- Docker + Docker Compose

### 1) Clone + install
```bash
git clone <your-repo-url> adpulse-aaas
cd adpulse-aaas
pnpm install
2) Configure env

Copy env templates:

cp .env.example .env
cp apps/api/env.example apps/api/.env
cp apps/worker/env.example apps/worker/.env
cp apps/web/env.example apps/web/.env.local
cp apps/mobile/env.example apps/mobile/.env
3) Start infra (Postgres + Redis + optional tools)
cd infra/docker
docker compose up -d
4) Run migrations

Choose one approach:

A) SQL migrations

pnpm db:migrate
pnpm db:seed

B) Supabase CLI (if used)

pnpm supabase:start
pnpm supabase:migrate
pnpm supabase:seed
5) Start apps

From repo root:

pnpm dev

Expected:

Web: http://localhost:3000

API: http://localhost:4000

Worker: background process consuming queues

Production Deployment
Recommended baseline

1 VPS (2vCPU/4GB RAM) for SMB workloads

Docker Compose

Caddy TLS

Nightly backups to S3/R2

Deploy steps (high level)

Provision VPS and install Docker

Set production env files (.env.prod)

Start stack:

docker compose -f infra/docker/compose.yaml up -d --build

Run migrations:

docker exec -it adpulse-api pnpm db:migrate

Configure DNS:

app.yourdomain.com → web

api.yourdomain.com → api

Lock down:

allowlist webhook endpoints

set rate limits

rotate secrets

Environment Variables
API (apps/api/.env)

PORT=4000

DATABASE_URL=postgresql://...

REDIS_URL=redis://...

JWT_PUBLIC_KEY=... or OIDC_ISSUER=...

WEBHOOK_SECRET_META=...

WEBHOOK_SECRET_PAYSTACK=...

MPESA_CONSUMER_KEY=...

MPESA_CONSUMER_SECRET=...

MPESA_SHORTCODE=...

MPESA_PASSKEY=...

WHATSAPP_PROVIDER=twilio|infobip|meta

WHATSAPP_API_KEY=...

LLM_PROVIDER=openai|...

LLM_MODEL=...

LLM_MAX_COST_PER_LEAD=...

Worker (apps/worker/.env)

DATABASE_URL=...

REDIS_URL=...

QUEUE_CONCURRENCY=...

SCHEDULE_PACING_MINUTES=30

SCHEDULE_REBALANCE_HOURS=6

SCHEDULE_FATIGUE_DAILY_AT=02:30

Web (apps/web/.env.local)

NEXT_PUBLIC_API_URL=https://api.yourdomain.com

NEXT_PUBLIC_APP_URL=https://app.yourdomain.com

Mobile (apps/mobile/.env)

EXPO_PUBLIC_API_URL=https://api.yourdomain.com

Webhook Integrations
M-Pesa (Daraja)

POST /webhooks/mpesa/callback

Required:

validate callback signature (provider-specific)

idempotency using provider_event_id

write to payments then trigger booking confirmation flow

Paystack / Stripe

POST /webhooks/paystack

POST /webhooks/stripe

Always:

verify signature

idempotency

log raw payload reference for audits

WhatsApp

POST /webhooks/whatsapp/inbound

Flow engine:

reads conversation state

emits next action (message, calendar query, deposit request)

logs every step in wa_flow_runs

Database and Migrations

Migrations live in:

database/migrations/*.sql

Key objects:

truth layer: events

ads: campaigns, experiments, ad_variations, ad_metrics_hourly

CRM: leads, lead_activity

bookings: resources, availability_blocks, bookings

payments: payments, deposit_policies

governance: decision_logs, audit_logs

dashboards: materialized views for <2s reads

Security Model

RBAC: owner, marketer, agent, finance, viewer

All write endpoints require auth + org scoping

Secrets never stored in DB unencrypted

Webhook endpoints:

signature verification

strict idempotency

rate limiting

Audit trail:

every automated budget/bid/pause action logged

rollback supported

Observability

Minimum:

structured logs (JSON)

request IDs

job IDs for worker tasks

Optional:

Prometheus + Grafana (infra/observability)

Sentry for error reporting

Testing

Unit tests:

packages/decision-engine (must be deterministic)

connectors with mocked providers

Integration tests:

webhook idempotency

booking + deposit confirmation pipeline

E2E:

orchestrator launch → experiment created → worker deploys → metrics ingested

Commands:

pnpm lint
pnpm typecheck
pnpm test
pnpm test:e2e
Roadmap
Phase 0 (7–10 days)

Events truth layer + CRM + inquiry capture + basic dashboard

Phase 1 (2–4 weeks)

Booking funnel + deposit verification + reminders + review automation

Phase 2 (4–6 weeks)

Orchestrator v1: A/B/C deploy + guardrails + auto pacing/rebalance

Phase 3

Predictive scaling (24–72h) + fatigue engine + cross-platform budget optimizer

License

Choose based on your commercial plan:

OSS-first commercial friendly: Apache-2.0

If you want to restrict SaaS reselling: consider AGPL (get legal advice)

Contact / Maintainers

Maintainer: <Your Name / Team>

Ops: see docs/runbooks/
