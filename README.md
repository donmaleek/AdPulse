# AAIS — Autonomous Ad Intelligence System (Africa)
**AAIS** is a revenue recovery + automation OS for African businesses to **run ads like a pro without hiring an agency** by combining:

**Attention Engine → Lead Engine → Booking/Payment Engine → Retention Engine**  
All measured end-to-end using **server-side truth** (not only platform pixels).

---

## What AAIS Solves (African reality)
Most businesses in Kenya/Africa:
- don’t track properly (no unified truth)
- don’t automate testing (no A/B/C discipline)
- don’t use predictive scaling (no pacing / guardrails)
- don’t centralize data (WhatsApp + payments + bookings live in different places)
- lose revenue on follow-ups, booking chaos, and deposit enforcement

AAIS fixes this by centralizing:
- ads + experiments
- leads + CRM
- WhatsApp booking bot + reminders
- M-Pesa deposits + verification
- review + repeat customer automation
- dashboards + decision engine

---

## Core Capabilities
### 1) AI Ad Orchestrator (No vague claims)
Given an offer, AAIS:
1. Computes **profit intelligence** (margin, Max CPA, capacity constraints)
2. Creates **3 test variations** (A/B/C) with enforceable templates
3. Deploys to **Meta / Google / TikTok** (connectors)
4. Monitors performance and **reallocates budget** on schedule
5. Enforces **guardrails** (caps, auto-pause, approvals, rollback)

### 2) “Run Ads Like a Pro (No Agency)” Mode
A guided wizard:
Offer → Margin → Tracking check → Landing QA → A/B/C launch → Daily report → Scale checklist

### 3) WhatsApp Booking Bot + Funnel
- Inquiry capture link/QR/widget → WhatsApp thread state machine
- Availability check (calendar/resources)
- Deposit request via **M-Pesa STK**
- Auto-verification → confirmation → reminders → no-show flow
- Review request + winback sequences

### 4) CRM + Lead Tracking
- Pipeline stages (standardized)
- Lead scoring (rules-first)
- Follow-up automation + broadcasts

### 5) Analytics Dashboard
- Executive: Profit, MER, CAC/CPA, pipeline value
- Ads: CTR/CPC/CPA, fatigue, experiments status
- Ops: response time, booking conversion, deposit compliance
- Retention: repeat rate, review velocity

---

## Architecture (Traffic → Data → Intelligence → Decision → Execution)
### Execution Surfaces
- Meta Ads (FB/IG), Google Ads, TikTok Ads
- WhatsApp, Web landing pages, Email/SMS (optional)
- Google Business Profile (SEO + reviews)

### Data Collection Layer (Single Source of Truth)
**Preferred:** server-side event ingestion + platform server APIs
- Meta CAPI, Google Enhanced Conversions
**Also:** pixel events for redundancy
**Webhooks:** M-Pesa Daraja, WhatsApp provider, calendar, Paystack/Stripe (optional)

### Intelligence Layer (specific jobs)
- Margin & Max CPA calculation
- Experiment evaluation (Bayesian/sequential)
- Fatigue + anomaly detection
- Short-horizon forecasting (24–72h) when enough data

### Decision Engine
- pacing, budget split, keep/kill/scale
- anomaly triggers
- attribution reconciliation (server truth vs platform)

### Execution Engine
- campaign CRUD + adjustments
- WhatsApp flows
- deposit enforcement + calendar confirmation
- review requests + winbacks
- audit logs + rollback

---

## Monorepo Layout

apps/web - Next.js admin + wizards
apps/api - Core API (NestJS/Fastify) + webhooks
apps/worker - BullMQ workers (pacing, reminders, reporting)
apps/mobile - Expo RN client (ops + quick actions)
apps/landing - Optional marketing site

packages/shared - shared types + zod + constants
packages/decision-engine - deterministic decision + stats
packages/connectors - Meta/Google/TikTok/WhatsApp/M-Pesa wrappers
packages/ui - shared UI kit
packages/config - lint/build configs

infra/ - docker compose, Caddy, scripts
docs/ - architecture + runbooks + SLAs


---

## Tech Stack (Production)
### Frontend
- Next.js 14 + TypeScript
- Tailwind + shadcn/ui
- Playwright (e2e), Vitest/Jest (unit)

### Backend
- Node.js + NestJS (recommended) or Fastify
- BullMQ + Redis for queues and schedules
- OpenAPI (Swagger)
- RBAC + audit logs

### Database
- **MongoDB Atlas** (primary)
  - TTL indexes (tentative bookings, OTPs, stale reservations)
  - Time-series collections (hourly metrics)
  - Atlas Search (lead & conversation search)
  - Change Streams (reactive workflows)

### Messaging
- WhatsApp Business API via BSP (Meta Cloud API / Twilio / Infobip / Africa’s Talking)
- Provider = transport; bot logic = AAIS

### Storage
- Cloudflare R2 (or S3-compatible)

### Observability
- Structured logs
- Metrics optional: Prometheus/Grafana
- Tracing optional: OpenTelemetry

---

## SLOs, Latency Targets, Cost Ceilings
### Latency (p95)
- Webhook ingestion: < 500ms
- Event write: < 150ms
- WhatsApp bot response:
  - deterministic: < 800ms
  - LLM-backed: < 2.5s
- Dashboard common queries: < 2s (pre-aggregations)

### Decision cadence
- anomaly checks: every 30 min
- rebalance: every 6–12 hours
- daily report generation: < 3 minutes

### Cost ceilings
- infra baseline: $20–$60/mo SMB production
- LLM usage: cap by **$ per lead-equivalent**
- Creative generation: templates by default; paid gen only for winners

---

## Environment Variables
Create `.env.local` from `.env.local.example`.

### Core
- `NODE_ENV=production|development`
- `APP_URL=https://...`
- `API_URL=https://...`
- `JWT_SECRET=...`
- `JWT_REFRESH_SECRET=...`
- `ENCRYPTION_KEY=...` (32 bytes for AES-256-GCM)

### MongoDB Atlas
- `MONGODB_URI=mongodb+srv://...`
- `MONGODB_DB=aais`
- `MONGODB_APP_NAME=aais-api`

### Redis / BullMQ
- `REDIS_URL=redis://...`
- `QUEUE_PREFIX=aais`

### Storage (R2/S3)
- `S3_ENDPOINT=...`
- `S3_BUCKET=...`
- `S3_ACCESS_KEY=...`
- `S3_SECRET_KEY=...`

### WhatsApp Provider
- `WHATSAPP_PROVIDER=meta|twilio|infobip|africastalking`
- Provider-specific keys…
- `WHATSAPP_WEBHOOK_SECRET=...`

### M-Pesa Daraja
- `MPESA_CONSUMER_KEY=...`
- `MPESA_CONSUMER_SECRET=...`
- `MPESA_SHORTCODE=...`
- `MPESA_PASSKEY=...`
- `MPESA_ENV=sandbox|production`
- `MPESA_WEBHOOK_SECRET=...`

### Ad Platforms (connectors)
- `META_APP_ID=...`
- `META_APP_SECRET=...`
- `META_ACCESS_TOKEN=...`
- `GOOGLE_ADS_CLIENT_ID=...`
- `GOOGLE_ADS_CLIENT_SECRET=...`
- `GOOGLE_ADS_DEVELOPER_TOKEN=...`
- `TIKTOK_APP_ID=...`
- `TIKTOK_APP_SECRET=...`

---

## Local Development (Docker-first)
### 1) Install
```bash
pnpm install
2) Start infra (local Redis + optional local Mongo for dev)
cd infra/docker
docker compose up -d

For production, use MongoDB Atlas. For local dev, you can run Mongo locally or point to Atlas.

3) Run API / Worker / Web
pnpm dev
# or individually:
pnpm --filter @aais/api dev
pnpm --filter @aais/worker dev
pnpm --filter @aais/web dev
pnpm --filter @aais/mobile dev
4) Create Mongo indexes (required)
pnpm --filter @aais/api run create-indexes
Deployment (Single VPS, production-grade)

Recommended: a small VPS running Docker + Caddy (TLS).

apps/api behind Caddy

apps/worker as separate service

apps/web static/SSR deployment (Vercel/CF Pages) or containerized

Redis on VPS (or Upstash)

MongoDB Atlas managed

Must-have production controls

Spend caps and approval thresholds

Idempotent webhooks

Audit log for every automated change

Rollback to last stable configuration

Security Model

RBAC roles: owner, marketer, agent, finance, viewer

All money-path actions require:

idempotency key

signature verification

audit log entry

Secrets stored in env + encrypted store if needed

PII encryption at rest for sensitive fields (AES-256-GCM)

Key Workflows (End-to-End)

Ad → Lead → WhatsApp → Booking → Deposit → Confirmation

A/B/C Test → Keep/Kill/Scale → Rotate on fatigue

Review request → resolve negatives → winback/loyalty automation

License

See LICENSE.


---

# MongoDB Atlas Schema (Production-Grade)

Below is a **practical Atlas schema** with:
- **collections**
- **document shapes**
- **indexes (incl TTL)**
- **collection validators**
- **time-series metrics**
- **idempotency + audit**

> Use **Atlas** for production; optionally mirror a local Mongo in docker for dev.

---

## 1) Naming & Conventions
- Database: `aais`
- Collections: `snake_case`
- IDs: use Mongo `_id` as `ObjectId`, plus `org_id` for multi-tenancy
- All writes include: `created_at`, `updated_at`
- Use “soft delete” where needed: `deleted_at`

---

## 2) Collections (Core)

## 2.1 `orgs`
**Purpose:** Multi-tenant boundary.

**Document**
```json
{
  "_id": { "$oid": "..." },
  "name": "Shanzu Beachfront Apartments",
  "country": "KE",
  "currency": "KES",
  "timezone": "Africa/Nairobi",
  "settings": {
    "default_language": "en",
    "max_daily_spend_cap": 50000,
    "require_approval_over_pct": 30
  },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ country: 1 }

{ name: 1 }

2.2 users
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "email": "admin@company.com",
  "phone_e164": "+2547xxxxxxx",
  "name": "Keith",
  "roles": ["owner"],
  "status": "active",
  "last_login_at": { "$date": "..." },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

Unique: { org_id: 1, email: 1 }

{ org_id: 1, status: 1 }

2.3 offers

Purpose: the “source of truth” for margin + constraints.

{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "type": "service", 
  "name": "Teeth Whitening",
  "price": { "amount": 8500, "currency": "KES" },
  "costs": {
    "cogs": 1200,
    "ops": 800,
    "delivery": 0,
    "payment_fee_pct": 1.5,
    "refund_rate_pct": 2.0
  },
  "capacity": {
    "mode": "appointments",
    "units_per_day": 12,
    "resource_ids": ["chair_1", "chair_2"]
  },
  "profit_intel": {
    "contribution_margin": 5600,
    "max_cpa": 3360,
    "safety_factor": 0.6
  },
  "targeting": {
    "countries": ["KE"],
    "cities": ["Nairobi"],
    "languages": ["en", "sw"]
  },
  "assets": { "primary_asset_id": { "$oid": "..." } },
  "status": "active",
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, status: 1 }

{ org_id: 1, type: 1 }

Validator (MongoDB JSON Schema)

db.createCollection("offers", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["org_id","type","name","price","costs","status","created_at","updated_at"],
      properties: {
        org_id: { bsonType: "objectId" },
        type: { enum: ["product","service","event","property","clinic_appointment","coaching"] },
        name: { bsonType: "string", minLength: 2 },
        price: {
          bsonType: "object",
          required: ["amount","currency"],
          properties: {
            amount: { bsonType: ["int","long","double"], minimum: 0 },
            currency: { bsonType: "string", minLength: 3, maxLength: 3 }
          }
        },
        status: { enum: ["active","paused","archived"] }
      }
    }
  }
})
2.4 assets
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "kind": "image",
  "storage": {
    "provider": "r2",
    "bucket": "aais-assets",
    "key": "orgs/<orgId>/assets/<file>.jpg",
    "content_type": "image/jpeg",
    "size_bytes": 1827723,
    "sha256": "..."
  },
  "tags": ["creative","before_after"],
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, kind: 1 }

{ "storage.sha256": 1 } (dedupe)

2.5 campaigns
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "offer_id": { "$oid": "..." },
  "platforms": ["meta","google","tiktok"],
  "objective": "leads",
  "status": "active",
  "budgets": {
    "daily_cap": 5000,
    "monthly_cap": 100000,
    "approval_required_over_pct": 30
  },
  "guardrails": {
    "auto_pause_cpa_multiplier": 1.3,
    "min_conversions_before_action": 3,
    "cooldown_minutes": 180
  },
  "platform_refs": {
    "meta": { "ad_account_id": "act_...", "campaign_id": "..." },
    "google": { "customer_id": "...", "campaign_id": "..." }
  },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, status: 1 }

{ org_id: 1, offer_id: 1 }

2.6 variations (A/B/C)
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "campaign_id": { "$oid": "..." },
  "key": "A",
  "angle": "direct_offer",
  "hook": "Get whiter teeth in 45 minutes",
  "creative_asset_id": { "$oid": "..." },
  "status": "testing",
  "deployed": {
    "meta": { "ad_id": "...", "adset_id": "..." },
    "google": { "asset_group_id": "..." }
  },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

Unique: { org_id: 1, campaign_id: 1, key: 1 }

{ org_id: 1, status: 1 }

2.7 events (Server Truth)

Purpose: unified event stream (page, lead, booking, payment).

{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "type": "lead_created",
  "ts": { "$date": "..." },
  "actor": { "kind": "lead", "id": { "$oid": "..." } },
  "attribution": {
    "utm": {
      "source": "facebook",
      "medium": "cpc",
      "campaign": "whitening_feb",
      "content": "varA"
    },
    "click_ids": { "fbclid": "...", "gclid": null, "ttclid": null }
  },
  "payload": { "lead_id": { "$oid": "..." }, "phone_e164": "+254..." },
  "ingest": { "ip": "1.2.3.4", "user_agent": "...", "sig_ok": true },
  "created_at": { "$date": "..." }
}

Indexes

{ org_id: 1, ts: -1 }

{ org_id: 1, type: 1, ts: -1 }

{ "attribution.click_ids.gclid": 1 }

{ "attribution.click_ids.fbclid": 1 }

For very high volumes, shard by org_id.

2.8 leads
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "source": { "channel": "meta", "campaign_id": { "$oid": "..." }, "variation": "A" },
  "contact": { "name": "Amina", "phone_e164": "+2547...", "email": null },
  "stage": "new",
  "score": 62,
  "assigned_to": { "$oid": "..." },
  "notes": [],
  "last_touch": { "ts": { "$date": "..." }, "type": "whatsapp_message" },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, stage: 1, updated_at: -1 }

Unique optional: { org_id: 1, "contact.phone_e164": 1 } (if your org wants 1 lead per phone)

2.9 conversations (WhatsApp state machine)
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "lead_id": { "$oid": "..." },
  "provider": "meta_cloud",
  "thread": { "wa_id": "2547...", "phone_e164": "+2547..." },
  "state": { "flow": "booking", "step": "awaiting_date", "context": {} },
  "last_message_at": { "$date": "..." },
  "flags": { "handoff_required": false },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, lead_id: 1 }

{ org_id: 1, "thread.wa_id": 1 }

2.10 resources (availability objects)
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "type": "room|chair|seat|staff",
  "name": "Room 203",
  "calendar": { "provider": "google", "calendar_id": "..." },
  "active": true,
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, active: 1 }

2.11 bookings (with TTL for tentative)
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "lead_id": { "$oid": "..." },
  "offer_id": { "$oid": "..." },
  "resource_id": { "$oid": "..." },
  "status": "tentative",
  "slot": { "start": { "$date": "..." }, "end": { "$date": "..." } },
  "deposit": { "required": true, "amount": 2500, "currency": "KES", "paid": false },
  "expires_at": { "$date": "..." },      // TTL target
  "confirmation": { "sent": false, "sent_at": null },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, status: 1, "slot.start": 1 }

{ org_id: 1, resource_id: 1, "slot.start": 1 }

TTL index: { expires_at: 1 } with expireAfterSeconds: 0

2.12 payments
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "provider": "mpesa_daraja",
  "direction": "inbound",
  "purpose": "deposit",
  "amount": { "value": 2500, "currency": "KES" },
  "status": "confirmed",
  "refs": {
    "provider_ref": "ws_CO_123...",
    "checkout_request_id": "...",
    "merchant_request_id": "..."
  },
  "matched": { "booking_id": { "$oid": "..." }, "lead_id": { "$oid": "..." } },
  "raw": { "daraja": { } },
  "received_at": { "$date": "..." },
  "created_at": { "$date": "..." }
}

Indexes

Unique: { org_id: 1, "refs.provider_ref": 1 }

{ org_id: 1, status: 1, received_at: -1 }

2.13 reviews
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "lead_id": { "$oid": "..." },
  "booking_id": { "$oid": "..." },
  "platform": "google",
  "request": { "sent_at": { "$date": "..." }, "link": "https://..." },
  "result": { "rating": 5, "comment": "..." },
  "routing": { "negative_flow_triggered": false },
  "created_at": { "$date": "..." },
  "updated_at": { "$date": "..." }
}

Indexes

{ org_id: 1, platform: 1, "result.rating": 1 }

{ org_id: 1, "request.sent_at": -1 }

2.14 audit_logs (non-negotiable)
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "actor": { "type": "user|system", "id": { "$oid": "..." } },
  "action": "campaign_budget_updated",
  "target": { "collection": "campaigns", "id": { "$oid": "..." } },
  "reason": { "rule": "rebalance_6h", "evidence": { "cpa": 980, "max_cpa": 1200 } },
  "before": { "daily_cap": 5000 },
  "after": { "daily_cap": 6500 },
  "created_at": { "$date": "..." }
}

Indexes

{ org_id: 1, created_at: -1 }

{ org_id: 1, action: 1, created_at: -1 }

2.15 idempotency_keys (webhooks + money paths)
{
  "_id": { "$oid": "..." },
  "org_id": { "$oid": "..." },
  "key": "mpesa:ws_CO_123...",
  "scope": "payments_webhook",
  "first_seen_at": { "$date": "..." },
  "expires_at": { "$date": "..." }
}

Indexes

Unique: { org_id: 1, key: 1 }

TTL: { expires_at: 1 } expireAfterSeconds: 0

3) Time-Series Metrics (Atlas)
3.1 ad_metrics_hourly (Time Series Collection)

Use MongoDB time-series for hourly metrics (cheap + fast for dashboards).

Create

db.createCollection("ad_metrics_hourly", {
  timeseries: {
    timeField: "ts",
    metaField: "meta",
    granularity: "hours"
  }
})

Doc

{
  "ts": { "$date": "2026-02-28T10:00:00Z" },
  "meta": {
    "org_id": { "$oid": "..." },
    "platform": "meta",
    "campaign_id": { "$oid": "..." },
    "variation_id": { "$oid": "..." }
  },
  "spend": 320,
  "impressions": 8400,
  "clicks": 210,
  "conversions": 4,
  "revenue": 0,
  "cpc": 1.52,
  "cpm": 38.1,
  "ctr": 0.025,
  "cpa": 80
}

Indexes

Mongo auto-indexes time-series internals; also add:

db.ad_metrics_hourly.createIndex({ "meta.org_id": 1, "meta.campaign_id": 1, ts: -1 })
4) Index Pack (Recommended)

Run once (your apps/api/scripts/create-indexes.ts should do this):

TTL

bookings.expires_at TTL

idempotency_keys.expires_at TTL

otp_codes.expires_at TTL (if you add OTP)

Multi-tenant filters

Most queries should start with org_id:

org_id + stage + updated_at

org_id + status + created_at

org_id + ts

5) Optional: Atlas Search (Leads + Conversations)

Create an Atlas Search index on:

leads.contact.name, leads.contact.phone_e164, leads.notes

conversations.thread.phone_e164, recent message text (if stored)
