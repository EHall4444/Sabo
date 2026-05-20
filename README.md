# Sabo

**Give everyday voters coordinated economic leverage against corporate lobbying.**

---

## The Problem

Corporations and billionaires effectively run government through lobbying and campaign contributions. Traditional civic engagement — voting, calling representatives — feels futile because the system is captured by money.

The only language corporations understand is money.

But individual consumers feel powerless. A $25 purchase decision doesn't move the needle alone. People need clarity on what companies are doing right now, a way to coordinate their spending power with others, and proof that their actions matter.

Existing tools don't solve this. Boycott apps like Buycott and Good On You use backward-looking, static data with no aggregate feedback. "Email your representative" tools don't work and users know it. Values-based shopping requires research, feels subjective, and doesn't target active harm.

**The insight:** What *does* work is hitting companies at their most vulnerable moment — when they're actively lobbying against something the public broadly supports.

---

## The Solution

SABO monitors active corporate lobbying against bipartisan, popular legislation. When you're about to give money to a flagged company, SABO alerts you — not with vague "this company is bad" messaging, but with specifics:

> *"This company is spending $3M this week to kill a bill 76% of Americans support."*

You log your action. You see your $25 become part of $47K in aggregate pressure. You watch the scoreboard move.

---

## How It Works

**Browser Extension (Chrome)**
- Detects when you land on a flagged company's site
- Shows a non-intrusive sidebar alert with plain-language context
- Lets you log one-click actions: switch subscriptions, move accounts, skip purchases
- Displays a live aggregate scoreboard

**Web Dashboard**
- Active campaign cards ranked by momentum and aggregate dollars shifted
- Leaderboard showing which companies are bleeding revenue fastest
- Deep-dive campaign pages with lobbying timelines, bill text, and FEC/LDA filings
- User profile tracking your personal impact and history

---

## Data Sources

SABO only uses public, auditable sources:

- **LDA Filings** — Senate Office of Public Records ([lda.senate.gov](https://lda.senate.gov)). Quarterly lobbying disclosures with client names, issues, and dollars spent.
- **FEC Records** — Federal Election Commission ([fec.gov](https://www.fec.gov/data/)). Corporate PAC donations, ~10 day lag.
- **OpenSecrets** — Cleaned FEC + lobbying data with programmatic API access.
- **Congress.gov** — Official bill tracking, status, and sponsors.
- **Public polling** — Pew Research, Gallup. Only campaigns with >60% public support are surfaced.

We don't catch dark money. We catch the big, documented campaigns where corporate pressure is highest and public pressure matters most. Everything is linked to its source filing.

---

## Project Status

**Pre-MVP — Active Development**

- [ ] Phase 1: Research & validate 3–5 launch campaigns (Weeks 1–2)
- [ ] Phase 2: Build automated LDA data pipeline (Weeks 3–4)
- [ ] Phase 3: Build Chrome extension — alert + action logging (Weeks 5–8)
- [ ] Phase 4: Build web dashboard — scoreboard + campaign pages (Weeks 9–12)
- [ ] Phase 5: Beta launch — 100 users (Week 13+)

---

## Tech Stack

- **Extension:** Chrome Manifest v3, React
- **Backend:** Node.js / Python, REST API
- **Frontend:** React, Recharts / D3
- **Real-time:** WebSocket
- **Database:** PostgreSQL (graduating from Airtable at MVP)

---

## MVP Success Criteria

- 1,000+ extension installs
- 500+ logged actions
- $100K+ aggregate dollars shifted
- 3–4 active campaigns live
- At least one media mention
- Users returning for multiple sessions

---

## What SABO Is Not

- Not partisan. Every campaign requires >60% public support and bipartisan polling validation.
- Not a data broker. User data is never sold.
- Not pay-to-play. Core functionality is free. No corporate sponsorships that create conflicts of interest.
- Not claiming to catch everything. Dark money PACs and shell companies exist. We show what's legally required to be disclosed.

---

## Contributing

SABO is in early ideation. If you're interested in contributing — data pipelines, extension development, campaign research, or UX — open an issue or reach out directly.

---

## Resources

| Source | URL |
|---|---|
| LDA Filings | https://lda.senate.gov |
| FEC Data | https://www.fec.gov/data/ |
| Congress.gov | https://www.congress.gov |
| OpenSecrets | https://www.opensecrets.org |
| Pew Research | https://www.pewresearch.org |
| Gallup | https://www.gallup.com |

---

*If the data is real-time, the scoreboard is visible, and corporate responses are fast enough to see — SABO makes corporate power feel beatable.*
