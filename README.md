# SABO

**A web-based consumer activism platform where users join groups organized around specific corporate grievances and coordinate financial and reputational pressure campaigns.**

---

## What It Does

SABO connects individual consumer actions into visible, aggregate economic pressure on corporations — anchored entirely to documented, verifiable public records.

Users join **groups** organized around specific corporate grievances (e.g., Amazon's labor practices, Cigna's coverage denials). They pick **tactics** from a curated library, take action, report back with proof, and watch their contribution reflected in the group's live scoreboard.

The core loop:

> **Grievance → Group → Tactic → Action → Report Back → Score → Escalation**

---

## Launch Campaigns

Four seeded campaigns go live at launch, each anchored to a primary source document (government filing, court judgment, or regulatory enforcement action):

- **Amazon**
- **Meta**
- **Cigna**
- **Wells Fargo**

---

## Key Features

**Group Pages**
Each group displays the documented grievance with a primary source link, active member count, live tactic feed, milestone tracker, and company acknowledgment status — all publicly viewable without an account.

**Tactic Library**
A curated set of legal individual actions, categorized by impact type: Revenue, Regulatory, Reputation, Capital, and Operations. Each tactic shows effort level, estimated corporate cost rating, usage count, community rating, and step-by-step instructions.

**Real-Time Action Feed**
A live feed of member actions — receipts, screenshots, confirmation posts — updated within 60 seconds of submission. Anonymized by default; public attribution is opt-in.

**Contribution Scoring**
Actions are weighted by tactic impact category. Users see their score broken down by group and category, with an opt-in leaderboard per group.

**Conglomerate Compass**
A publicly accessible ownership lookup tool. Search any company, see who owns it and what it owns, sourced from SEC EDGAR filings. Linked directly to active SABO groups where applicable.

**Company Acknowledgment Tracker**
Every group page shows whether the target company has responded: *No Response*, *Statement Issued*, or *Behavior Changed* — the visible win state of the platform.

**Group Proposals**
Logged-in users can propose new campaigns by submitting a company name, grievance, and verifiable primary source. Community upvotes auto-activate groups at a configurable threshold.

---

## Design Principles

**Bipartisan by Design** — Every grievance must be anchored to a verifiable primary source (government filing, court judgment, or regulatory enforcement action). Editorial opinion does not qualify. All copy uses neutral, factual language.

**Privacy First** — Account creation requires only an email and password. All actions are anonymized by default. User data is never sold, licensed, or transferred.

**Legal by Default** — Every tactic in the library is a legal action any individual can take alone. All Capital and Regulatory tactics pass legal review before publication.

**No Dark Money** — SABO uses only public, auditable data sources. Every claim links to its source record.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.11, FastAPI, SQLAlchemy (async), Alembic |
| Frontend | SvelteKit, TypeScript, adapter-node |
| Database | PostgreSQL 16 |
| Real-time | Server-Sent Events (SSE) + Redis pub/sub |
| Background jobs | Celery + Redis |
| File storage | Hetzner Object Storage (S3-compatible) |
| Auth | JWT (httpOnly cookie) + bcrypt |
| Corporate data | SEC EDGAR Exhibit 21 (ownership), OpenCorporates (planned) |
| Hosting | Hetzner VPS, Nginx, Cloudflare CDN |
| Analytics | Plausible (self-hosted) |

No AWS, Google Cloud, or Meta infrastructure at any layer.

---

## Project Status

**Pre-MVP — Active Development**

| Phase | Description | Status |
|---|---|---|
| 0 | Project scaffold, Docker Compose, Alembic | 🔲 |
| 1 | Auth, core models, seed data | 🔲 |
| 2 | **Core loop — MVP gate** (US1) | 🔲 |
| 3 | Tactic library filtering (US2) | 🔲 |
| 4 | Real-time action feed (US3) | 🔲 |
| 5 | Leaderboard + scoring display (US4) | 🔲 |
| 6 | Group proposals + validation (US5) | 🔲 |
| 7 | Conglomerate Compass + constellation (US6) | 🔲 |
| 8 | Acknowledgment tracker win state (US7) | 🔲 |
| 9 | Polish + production hardening | 🔲 |

The MVP gate is Phase 2: a first-time user must be able to complete the full core loop — land on a group page, join, pick a tactic, submit proof, and see their score update — in under 10 minutes without instructions.

---

## MVP Success Criteria

- First-time user completes the full core loop in under 10 minutes, no instructions
- All four seeded campaigns live with documented grievances and at least 3 pre-loaded tactics each
- Tactic Library contains at least 10 tactics across at least 3 impact categories
- Action feed, leaderboard, and acknowledgment tracker all reflect new data within 60 seconds
- Group pages load on mobile in under 3 seconds on a standard 4G connection
- 100% of grievance copy passes a partisan-language audit
- No user PII visible to other users without explicit opt-in

---

## Data Sources

| Source | Use |
|---|---|
| [SEC EDGAR Exhibit 21](https://www.sec.gov/cgi-bin/browse-edgar) | Corporate ownership structure (Conglomerate Compass, constellation detection) |
| [OpenCorporates](https://opencorporates.com) | Extended global/private company coverage (planned post-launch) |
| Government filings, court judgments, regulatory actions | Grievance primary sources — the only accepted documentation type |

---

## What SABO Is Not

- Not partisan. Every campaign requires a verifiable public record as its primary source.
- Not a data broker. User data is never sold or transferred.
- Not claiming to catch everything. Dark money PACs and shell companies exist. SABO surfaces documented, disclosed corporate activity where pressure is measurable.
- Not a petition site. The acknowledgment tracker — *No Response → Statement Issued → Behavior Changed* — is how SABO closes the loop.

---

## Contributing

SABO is in active pre-MVP development. If you're interested in contributing to the backend, frontend, tactic research, or grievance validation, open an issue to start the conversation.

All tactic contributors must sign a contributor agreement before submissions enter the review queue. All submitted tactics pass legal review before publication.
