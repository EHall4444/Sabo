# Tasks: SABO MVP Web Platform

**Input**: Design documents from `specs/002-mvp-web-platform/`

**Prerequisites**: plan.md ✅ | spec.md ✅ | research.md ✅ | data-model.md ✅ | contracts/api-v1.md ✅ | quickstart.md ✅

**Tests**: Not explicitly requested — no test tasks generated. Integration scenarios in quickstart.md serve as manual validation gates.

**Organization**: Tasks grouped by user story. Each story is independently deliverable.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies on incomplete tasks)
- **[Story]**: Which user story this task belongs to (US1–US7)

---

## Phase 1: Setup

**Purpose**: Project scaffolding, tooling, and local dev environment. No business logic.

- [ ] T001 Create backend/ directory structure per plan.md: src/{api/v1,models,services,workers,core}, migrations/, tests/{unit,integration,contract}
- [ ] T002 Create backend/pyproject.toml with dependencies: fastapi, uvicorn[standard], sqlalchemy[asyncio], alembic, asyncpg, redis, celery, sse-starlette, python-jose[cryptography], passlib[bcrypt], boto3, httpx, pydantic-settings
- [ ] T003 [P] Create backend/src/core/config.py — pydantic-settings Settings class reading: DATABASE_URL, REDIS_URL, SECRET_KEY, HETZNER_ACCESS_KEY, HETZNER_SECRET_KEY, HETZNER_BUCKET, HETZNER_ENDPOINT, EDGAR_USER_AGENT, ENVIRONMENT
- [ ] T004 [P] Create backend/.env.example with all required env vars (no values)
- [ ] T005 Create frontend/ directory structure: src/{routes,lib/{components,api,stores,types}}, static/
- [ ] T006 Create frontend/package.json with dependencies: @sveltejs/kit, @sveltejs/adapter-node, svelte, typescript, vite
- [ ] T007 [P] Create frontend/svelte.config.js configured with adapter-node
- [ ] T008 [P] Create frontend/vite.config.ts with TypeScript and Svelte plugin
- [ ] T009 Create docker-compose.yml with services: postgres (16-alpine), redis (7-alpine), with named volumes and health checks
- [ ] T010 [P] Create nginx/sabo.conf — reverse proxy to backend :8000 and frontend :3000; proxy_buffering off for /api/v1/groups/*/feed/stream SSE endpoints

---

## Phase 2: Foundational

**Purpose**: Shared infrastructure that ALL user stories depend on. Must complete before any story work begins.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T011 Create backend/src/core/database.py — async SQLAlchemy engine, async session factory, Base declarative class, get_db dependency
- [ ] T012 [P] Create backend/src/core/redis.py — Redis connection pool, get_redis dependency, publish() and subscribe() helpers for pub/sub
- [ ] T013 [P] Create backend/src/core/security.py — JWT creation/validation (python-jose), bcrypt password hashing/verification (passlib), token expiry constants (access: 15 min, refresh: 7 days)
- [ ] T014 Create backend/migrations/env.py — Alembic async configuration pointing to DATABASE_URL from config
- [ ] T015 Create initial Alembic migration in backend/migrations/versions/ covering ALL entities from data-model.md: companies, groups, grievances, constellations, ownership_records, tactics, tactic_group_links, actions, proof_artifacts, users, group_memberships, user_group_scores, milestones, acknowledgment_records, group_proposals, votes, venting_posts — with all indexes from data-model.md
- [ ] T016 Create backend/src/models/user.py — SQLAlchemy async model: id (UUID PK), email (unique), email_verified, password_hash, display_name, is_staff, notification_opt_in, created_at, last_active_at
- [ ] T017 [P] Create backend/src/models/company.py — SQLAlchemy async model: id, name (unique), slug (unique), ticker, cik, logo_url, created_at
- [ ] T018 Create backend/src/services/auth_service.py — register_user(), authenticate_user(), create_access_token(), create_refresh_token(), get_current_user() FastAPI dependency
- [ ] T019 Create backend/src/api/v1/auth.py — POST /api/v1/auth/register, /login, /logout, /refresh with rate limiting on /login (5 failed/15 min/IP)
- [ ] T020 Create backend/src/api/__init__.py and backend/src/api/v1/__init__.py — FastAPI APIRouter setup
- [ ] T021 Create backend/main.py — FastAPI app factory: CORS, API router include (/api/v1), lifespan handler (DB pool startup/shutdown)
- [ ] T022 [P] Create frontend/src/lib/types/api.ts — TypeScript interfaces matching all API response shapes from contracts/api-v1.md: User, Group, Grievance, Tactic, Action, GroupProposal, OwnershipRecord, etc.
- [ ] T023 [P] Create frontend/src/lib/stores/auth.ts — Svelte writable store for current user (User | null), login(), logout(), register() calling backend auth endpoints
- [ ] T024 Create frontend/src/lib/api/auth.ts — fetch wrappers for POST /auth/register, /login, /logout, /refresh with httpOnly cookie handling
- [ ] T025 Create frontend/src/routes/auth/login/+page.svelte — login form, calls auth store login(), redirects on success, mobile-first layout
- [ ] T026 [P] Create frontend/src/routes/auth/register/+page.svelte — registration form, email + password (≥12 chars), mobile-first layout

**Checkpoint**: Backend auth works end-to-end. Frontend can register and log in. All DB tables exist. Ready for user story implementation.

---

## Phase 3: User Story 1 — Complete the Core Loop (Priority: P1) 🎯 MVP Gate

**Goal**: A user can land on a seeded campaign page, join the group, pick a tactic, submit proof of action, and see their score update. The full core loop is completable for all four seeded campaigns.

**Independent Test**: Quickstart Scenario 1 — all 10 steps pass without assistance.

### Models

- [ ] T027 Create backend/src/models/group.py — SQLAlchemy async model: id, company_id (FK), slug (unique), status (enum: Draft/Active/Archived), is_seeded, constellation_role (enum: Parent/Subsidiary/null), member_count, created_at, activated_at; relationship to Company, Grievance
- [ ] T028 [P] Create backend/src/models/grievance.py — SQLAlchemy async model: id, group_id (unique FK), title, body, primary_source_url, primary_source_type (enum), primary_source_label, cached_source_snapshot, published_at, created_by_staff; immutability enforced in service layer
- [ ] T029 [P] Create backend/src/models/tactic.py — SQLAlchemy async model: id, slug (unique), title, description, impact_category (enum: Reputation/Revenue/Regulatory/Capital/Operations), routing_level (enum: Subsidiary/Parent), effort_level (enum: Low/Medium/High), point_value, corporate_cost_rating, usage_count, community_rating_sum, community_rating_count, status (enum: Draft/UnderReview/Active/Removed), legal_review_passed, contributor_id, contributor_agreement_signed_at, created_at, removed_at
- [ ] T030 [P] Create backend/src/models/tactic_group_link.py — SQLAlchemy async model: id, tactic_id (FK), group_id (FK), custom_title, custom_instructions, is_featured; unique constraint on (tactic_id, group_id)
- [ ] T031 [P] Create backend/src/models/action.py — SQLAlchemy async model: id, user_id (FK), tactic_id (FK), group_id (FK), score, attestation_type (enum), status (enum: Submitted/Flagged/Confirmed/Voided), is_public, submitted_at, confirmed_at, voided_at, void_reason; unique constraint on (user_id, tactic_id, group_id)
- [ ] T032 [P] Create backend/src/models/proof_artifact.py — SQLAlchemy async model: id, action_id (FK), artifact_type (enum: Image/ConfirmationText), storage_key, storage_url, confirmation_text, uploaded_at; check constraint: at least one of storage_key or confirmation_text non-null
- [ ] T033 [P] Create backend/src/models/group_membership.py — SQLAlchemy async model: id, user_id (FK), group_id (FK), leaderboard_opt_in, public_attribution_opt_in, joined_at; unique constraint on (user_id, group_id)
- [ ] T034 [P] Create backend/src/models/user_group_score.py — SQLAlchemy async model: id, user_id (FK), group_id (FK), total_score, score_by_category (JSONB), action_count, last_updated_at; unique constraint on (user_id, group_id)
- [ ] T035 [P] Create backend/src/models/milestone.py — SQLAlchemy async model: id, group_id (FK), label, threshold_category (enum, nullable), threshold_value, is_crossed, crossed_at
- [ ] T036 [P] Create backend/src/models/acknowledgment_record.py — SQLAlchemy async model: id, group_id (unique FK), status (enum: NoResponse/StatementIssued/BehaviorChanged, default NoResponse), source_url, source_label, updated_by (FK User), updated_at, notes

### Services

- [ ] T037 Create backend/src/services/storage_service.py — boto3 S3 client configured with Hetzner endpoint_url; upload_proof_artifact(file_bytes, content_type) → storage_key; get_presigned_url(storage_key) → cdn_url; delete_artifact(storage_key)
- [ ] T038 Create backend/src/services/scoring_service.py — compute_action_score(tactic): returns tactic.point_value; update_user_group_score(db, user_id, group_id, tactic): increments UserGroupScore.total_score and score_by_category[category]; void_action_score(db, action): decrements UserGroupScore; check_and_cross_milestones(db, group_id): queries Milestone rows and marks is_crossed + crossed_at when threshold met

### API Endpoints (US1)

- [ ] T039 [US1] Create backend/src/api/v1/groups.py — GET /api/v1/groups (list, public), GET /api/v1/groups/{slug} (detail with grievance, scores, milestones, acknowledgment, current_user membership state), POST /api/v1/groups/{slug}/join (auth required), PATCH /api/v1/groups/{slug}/membership (auth + membership required)
- [ ] T040 [US1] Create backend/src/api/v1/tactics.py — GET /api/v1/tactics (list, filterable by category and routing_level), GET /api/v1/tactics/{id} (detail), GET /api/v1/groups/{slug}/tactics (group-specific with TacticGroupLink customization applied; returns subsidiary_tactics and parent_level_tactics separately per FR-047)
- [ ] T041 [US1] Create backend/src/api/v1/actions.py — POST /api/v1/actions (auth + membership; multipart/form-data; enforces duplicate unique constraint with 409; validates tactic routing_level matches group role; calls scoring_service and storage_service; publishes Redis event for SSE fan-out); POST /api/v1/actions/{id}/flag
- [ ] T042 [US1] Create backend/src/api/v1/users.py — GET /api/v1/users/me, PATCH /api/v1/users/me, GET /api/v1/users/me/actions (paginated history)

### Seed Data

- [ ] T043 [US1] Create backend/src/scripts/seed.py — seeds all four seeded campaigns: companies (Amazon/Meta/Cigna/Wells Fargo with CIKs), groups (Active, is_seeded=True), grievances (factual body + primary_source_url sourced to real public regulatory filings), acknowledgment_records (NoResponse), milestones; seeds 10–15 tactics across ≥3 categories (Revenue, Regulatory, Reputation minimum) with routing_level set; creates TacticGroupLinks with campaign-specific titles (e.g., "Cancel Amazon Prime")

### Frontend (US1)

- [ ] T044 [US1] Create frontend/src/lib/api/groups.ts — fetch wrappers for GET /groups, GET /groups/{slug}, POST /groups/{slug}/join, PATCH /groups/{slug}/membership
- [ ] T045 [P] [US1] Create frontend/src/lib/api/tactics.ts — fetch wrappers for GET /tactics, GET /tactics/{id}, GET /groups/{slug}/tactics
- [ ] T046 [P] [US1] Create frontend/src/lib/api/actions.ts — fetch wrapper for POST /actions (multipart/form-data), POST /actions/{id}/flag
- [ ] T047 [US1] Create frontend/src/routes/+page.svelte — home page: campaign directory grid (GET /groups), mobile-first card layout, links to individual group pages
- [ ] T048 [US1] Create frontend/src/routes/groups/[slug]/+page.server.ts — SSR data load: GET /groups/{slug} on server, returns group data for hydration
- [ ] T049 [US1] Create frontend/src/routes/groups/[slug]/+page.svelte — group detail page: GrievancePanel, member count, AcknowledgmentTracker, MilestoneTracker, TacticLibrary (filtered to this group), action submission CTA, mobile-first layout
- [ ] T050 [P] [US1] Create frontend/src/lib/components/GrievancePanel.svelte — displays grievance title, body (markdown rendered), primary source link with external link indicator; neutral factual styling
- [ ] T051 [P] [US1] Create frontend/src/lib/components/TacticCard.svelte — displays tactic title, impact category badge, effort level pill, corporate cost rating (1–5), usage count, community rating stars, point value; "Take Action" CTA button
- [ ] T052 [P] [US1] Create frontend/src/lib/components/TacticLibrary.svelte — renders subsidiary_tactics list + collapsible parent_level_tactics section ("Available at [Parent] level" with link) per FR-047; no filtering yet (added in US2)
- [ ] T053 [US1] Create frontend/src/lib/components/ProofUpload.svelte — file input (JPEG/PNG ≤10 MB) or confirmation text textarea; shows preview of selected image; submits multipart form to POST /actions
- [ ] T054 [P] [US1] Create frontend/src/lib/components/AcknowledgmentTracker.svelte — displays status badge (NoResponse/StatementIssued/BehaviorChanged), source link, days since launch; win state styling for BehaviorChanged
- [ ] T055 [P] [US1] Create frontend/src/lib/components/MilestoneTracker.svelte — displays milestone list with progress indicators; celebration styling for is_crossed milestones

**Checkpoint**: Quickstart Scenario 1 fully passable. All four seeded campaigns viewable and joinable. Actions submittable with proof. Scores update. This is the MVP gate — stop and validate before proceeding.

---

## Phase 4: User Story 2 — Browse and Execute Tactics from the Library (Priority: P2)

**Goal**: Full filterable Tactic Library with complete detail cards, category filtering, community ratings, and routing level enforcement.

**Independent Test**: Quickstart Scenario 2 — library filters correctly; routing level mismatch returns 400.

### API Endpoints (US2)

- [ ] T056 [US2] Add category and routing_level query param filtering to GET /api/v1/tactics in backend/src/api/v1/tactics.py (already scaffolded in T040; ensure filters are applied in query layer not in-memory)
- [ ] T057 [US2] Create backend/src/api/v1/tactics.py — POST /api/v1/tactics/{id}/rate: auth required, user must have at least one Confirmed action for this tactic (check Action table), one rating per user per tactic (unique constraint), updates community_rating_sum and community_rating_count on Tactic

### Frontend (US2)

- [ ] T058 [US2] Update frontend/src/lib/components/TacticLibrary.svelte — add category filter buttons (Reputation/Revenue/Regulatory/Capital/Operations), routing level toggle; filter state drives API call to GET /groups/{slug}/tactics with query params
- [ ] T059 [P] [US2] Create frontend/src/routes/tactics/+page.svelte — standalone Tactic Library page (not group-specific): loads GET /tactics with filter params, displays TacticCard grid, mobile-first layout
- [ ] T060 [P] [US2] Update frontend/src/lib/components/TacticCard.svelte — add full detail expansion (step-by-step instructions rendered as markdown, legal review status badge for Capital/Regulatory tactics, contributor attribution if third-party); add star rating submission UI (visible only after user has executed tactic)
- [ ] T061 [US2] Add routing_level_mismatch validation to POST /api/v1/actions in backend/src/api/v1/actions.py — verify tactic.routing_level matches group.constellation_role (Subsidiary tactic → group must not be a Parent-only group; Parent tactic → group must be a Parent); return 400 ROUTING_LEVEL_MISMATCH if violated

**Checkpoint**: Tactic Library fully filterable. Routing level validation enforced at API layer. Detail cards show all fields from contracts/api-v1.md.

---

## Phase 5: User Story 3 — Real-Time Action Feed (Priority: P3)

**Goal**: Live action feed on group pages updates within 60 seconds of any new action without page refresh. Venting feed is separate.

**Independent Test**: Quickstart Scenario 3 — two SSE clients both receive new_action event within 60 seconds; venting post does not appear on action board.

### Models

- [ ] T062 Create backend/src/models/venting_post.py — SQLAlchemy async model: id, group_id (FK), user_id (FK), body (max 1000 chars), is_public, posted_at

### Services

- [ ] T063 Create backend/src/services/feed_service.py — publish_action_event(redis, group_id, action_data): publishes JSON to Redis channel f"feed:{group_id}"; publish_score_update(redis, group_id, category, new_total): publishes score_update event; subscribe_to_feed(redis, group_id): returns async generator of events for SSE handler

### API Endpoints (US3)

- [ ] T064 [US3] Create backend/src/api/v1/stream.py — GET /api/v1/groups/{slug}/feed/stream: public SSE endpoint using sse-starlette EventSourceResponse; subscribes to Redis channel f"feed:{group_slug}"; yields new_action and score_update events with event IDs; handles client disconnect cleanup
- [ ] T065 [P] [US3] Add GET /api/v1/groups/{slug}/feed to backend/src/api/v1/groups.py — paginated action feed (page, page_size, category filter); joins Action + Tactic + ProofArtifact; applies anonymization (attribution = display_name if is_public=true AND GroupMembership.public_attribution_opt_in=true, else anonymized label); returns proof_thumbnail_url from storage_service.get_presigned_url()
- [ ] T066 [P] [US3] Add POST + GET /api/v1/groups/{slug}/venting to backend/src/api/v1/groups.py — venting posts served from separate endpoint; never included in /feed response or SSE stream
- [ ] T067 [US3] Update backend/src/api/v1/actions.py POST /actions — after persisting Action and updating UserGroupScore, call feed_service.publish_action_event() and feed_service.publish_score_update(); also call scoring_service.check_and_cross_milestones() and publish milestone_crossed event if threshold crossed

### Frontend (US3)

- [ ] T068 [US3] Create frontend/src/lib/stores/feed.ts — Svelte writable store for feed items; connect() opens EventSource to /api/v1/groups/{slug}/feed/stream; on new_action event prepends item to store; on score_update event updates group score display; disconnect() closes EventSource
- [ ] T069 [US3] Create frontend/src/lib/components/ActionFeed.svelte — subscribes to feed store on mount; renders ActionFeedItem list; shows "No actions yet this week — be the first" when feed is empty and last action > 7 days ago; mobile-first layout
- [ ] T070 [P] [US3] Create frontend/src/lib/components/ActionFeedItem.svelte — displays attribution label (anonymized or display_name), tactic name, category badge, score earned, timestamp, proof_thumbnail_url (inline thumbnail; click opens full image); mobile-first
- [ ] T071 [P] [US3] Update frontend/src/routes/groups/[slug]/+page.svelte — add ActionFeed component (action board tab/section) and venting surface (separate section below fold, styled differently); wire up feed store for live updates

**Checkpoint**: Quickstart Scenario 3 passes. Feed updates live. Venting surface visually separated and never contaminates action board.

---

## Phase 6: User Story 4 — Contribution Score and Leaderboard (Priority: P4)

**Goal**: Users can view their personal score breakdown and opt into a per-group leaderboard. Leaderboard updates within 60 seconds.

**Independent Test**: Quickstart Scenario 6 — score breakdown visible on profile; leaderboard opt-out immediately removes user from rankings.

### API Endpoints (US4)

- [ ] T072 [US4] Add GET /api/v1/groups/{slug}/leaderboard to backend/src/api/v1/groups.py — queries UserGroupScore WHERE leaderboard_opt_in=true AND public_attribution_opt_in=true for this group; orders by total_score DESC; paginates; returns rank, display_name, total_score, action_count, score_by_category
- [ ] T073 [P] [US4] Update backend/src/services/feed_service.py — add publish_leaderboard_update(redis, group_id, user_rank_data): publishes leaderboard_update SSE event; call after scoring_service.update_user_group_score() when user is leaderboard-opted-in

### Frontend (US4)

- [ ] T074 [US4] Create frontend/src/lib/components/Leaderboard.svelte — fetches GET /groups/{slug}/leaderboard; displays ranked list with display_name, total_score, score_by_category breakdown; updates on leaderboard_update SSE event from feed store; opt-in/opt-out toggle calls PATCH /groups/{slug}/membership; hides immediately on opt-out without page reload
- [ ] T075 [P] [US4] Create frontend/src/lib/components/ScoreBreakdown.svelte — displays user's score by category as labeled bars (Revenue: N pts, Regulatory: N pts, etc.); used on both group page (group-scoped) and profile page (cross-group total)
- [ ] T076 [US4] Create frontend/src/routes/profile/+page.svelte — auth-gated; loads GET /users/me and GET /users/me/actions; displays ScoreBreakdown (total + by category), action history list (group, tactic, score, date, proof thumbnail), display_name edit field, notification_opt_in toggle; mobile-first layout
- [ ] T077 [P] [US4] Update frontend/src/routes/groups/[slug]/+page.svelte — add Leaderboard component below ActionFeed; add ScoreBreakdown widget for current user's group-scoped score (visible only when authenticated and joined)

**Checkpoint**: Quickstart Scenario 4 and 6 pass. Leaderboard live-updates. Profile page shows full score history.

---

## Phase 7: User Story 5 — Propose and Validate a New Group (Priority: P5)

**Goal**: Users can propose new groups with primary source validation. Community upvotes auto-activate groups at threshold. Duplicate detection prevents redundant proposals.

**Independent Test**: Quickstart Scenario 4 steps 1–4 (proposal flow without constellation).

### Models

- [ ] T078 Create backend/src/models/group_proposal.py — SQLAlchemy async model: id, proposer_id (FK), company_name, company_cik, grievance_description, primary_source_url, primary_source_type (enum), upvote_count, upvote_threshold (default 50), status (enum: Pending/Approved/Rejected), constellation_parent_group_id (FK nullable), rejection_reason, submitted_at, resolved_at, resulting_group_id (FK nullable)
- [ ] T079 [P] Create backend/src/models/vote.py — SQLAlchemy async model: id, proposal_id (FK), voter_id (FK), voted_at; unique constraint on (proposal_id, voter_id)

### Services

- [ ] T080 Create backend/src/services/proposal_service.py — check_duplicate(db, company_name): case-insensitive match against active Group company names; create_proposal(db, data): saves GroupProposal; process_upvote(db, proposal_id, voter_id): adds Vote, increments upvote_count, calls auto_activate() if threshold met; auto_activate(db, proposal): creates Company + Group + Grievance + AcknowledgmentRecord from proposal data, sets proposal.resulting_group_id, notifies proposer; reject_proposal(db, proposal_id, reason): sets status=Rejected with reason

### API Endpoints (US5)

- [ ] T081 [US5] Create backend/src/api/v1/proposals.py — POST /api/v1/proposals (auth required; validates primary_source_type ≠ Editorial; calls check_duplicate(); returns 409 with existing group slug if duplicate; saves proposal; returns constellation_offer if EDGAR detects subsidiary — wired in Phase 8, returns null for now); GET /api/v1/proposals (public, paginated, filterable by status); POST /api/v1/proposals/{id}/upvote (auth required; enforces one-vote-per-user; triggers auto_activate when threshold met; returns 409 if already voted or proposal not Pending)

### Frontend (US5)

- [ ] T082 [US5] Create frontend/src/routes/proposals/+page.svelte — public list of Pending proposals; displays company name, grievance summary, upvote count vs threshold, days since submitted; "Upvote" button (auth required); mobile-first layout
- [ ] T083 [P] [US5] Create frontend/src/routes/proposals/new/+page.svelte — auth-gated proposal form: company name, grievance description (markdown textarea), primary source URL, primary source type selector (GovernmentFiling/CourtJudgment/RegulatoryAction/CongressionalTestimony); client-side validation; on submit calls POST /proposals; on 409 duplicate, shows link to existing group; on success shows proposal page with upvote CTA; on constellation_offer (Phase 8), shows attachment prompt

**Checkpoint**: Full proposal lifecycle works. Duplicate detection surfaces existing groups. Auto-activation creates group at upvote threshold and notifies proposer.

---

## Phase 8: User Story 6 — Conglomerate Compass (Priority: P6)

**Goal**: Any visitor can search for a company's ownership structure, see parent/subsidiary relationships sourced from SEC EDGAR, and discover linked SABO groups. Constellation detection wires into the proposal flow.

**Independent Test**: Quickstart Scenario 5 — Amazon search returns Whole Foods as subsidiary with SABO group link; unknown company returns gracefully; force-refresh is async.

### Models

- [ ] T084 Create backend/src/models/ownership_record.py — SQLAlchemy async model: id, company_id (unique FK), parent_company_name, parent_company_cik, subsidiaries (JSONB array of {name, jurisdiction, cik?}), data_source (enum: EDGAR/OpenCorporates/Both), edgar_filing_url, fetched_at, expires_at (fetched_at + 30 days), is_stale (computed: expires_at < now())
- [ ] T085 [P] Create backend/src/models/constellation.py — SQLAlchemy async model: id, parent_group_id (FK), subsidiary_group_id (unique FK), created_at, created_by_staff; check constraint: parent_group_id ≠ subsidiary_group_id

### Services

- [ ] T086 Create backend/src/services/edgar_service.py — EdgarDataSource implementing OwnershipDataSource interface; lookup(company_name): searches EDGAR EFTS (efts.sec.gov) for most recent 10-K filing by company name or CIK; fetches filing index to locate Exhibit 21 document; parses Exhibit 21 HTML/text to extract subsidiary list (company names + jurisdiction); returns OwnershipResult; enforces 8 req/s rate limit; User-Agent header set to EDGAR_USER_AGENT config value (required by SEC)
- [ ] T087 Create backend/src/services/compass_service.py — CompassService orchestrating OwnershipDataSource adapters; lookup(company_name) → fetches from enabled sources (EDGAR only for MVP), merges, caches as OwnershipRecord (30-day TTL); get_cached(company_id) → returns OwnershipRecord if not stale; refresh(company_id) → invalidates cache, re-fetches; inject_sabo_links(ownership_result) → annotates subsidiaries and parent with matching active Group slugs

### Celery Tasks

- [ ] T088 Create backend/src/workers/ownership_tasks.py — Celery app configured with Redis broker; fetch_ownership_async.delay(company_name, proposal_id): calls compass_service.lookup(), caches result, updates GroupProposal with constellation_offer if parent detected; refresh_ownership_async.delay(company_id): calls compass_service.refresh()

### API Endpoints (US6)

- [ ] T089 [US6] Create backend/src/api/v1/compass.py — GET /api/v1/compass/search?q= (public; calls compass_service.lookup() or returns cached; injects sabo_links; includes staleness notice in response if is_stale; returns empty results gracefully if not found); GET /api/v1/compass/companies/{cik} (public; returns full OwnershipRecord by CIK)
- [ ] T090 [US6] Create backend/src/api/v1/admin.py — POST /api/v1/admin/ownership-records/{company_id}/refresh (is_staff required; enqueues refresh_ownership_async Celery task; returns 202 with task_id); GET /api/v1/admin/flags; PATCH /api/v1/admin/actions/{id}/status

### Constellation Integration

- [ ] T091 [US6] Update backend/src/api/v1/proposals.py POST /proposals — after saving proposal, enqueue fetch_ownership_async Celery task; if OwnershipRecord already cached and parent detected, return constellation_offer in response immediately
- [ ] T092 [US6] Update backend/src/services/proposal_service.py auto_activate() — when constellation_parent_group_id is set on the proposal, create Constellation record linking new subsidiary group to parent; update parent group's constellation_role=Parent and new group's constellation_role=Subsidiary
- [ ] T093 [US6] Update backend/src/api/v1/groups.py GET /groups/{slug} — when group is a Parent, query Constellation table for subsidiaries; compute subsidiary_rollup scores (sum UserGroupScore per subsidiary group); return both direct scores and labeled subsidiary_rollup per FR-015c; when group is a Subsidiary, include parent group link in response

### Frontend (US6)

- [ ] T094 [US6] Create frontend/src/lib/api/compass.ts — fetch wrappers for GET /compass/search, GET /compass/companies/{cik}
- [ ] T095 [P] [US6] Create frontend/src/lib/components/CompassSearch.svelte — search input with debounce; calls GET /compass/search; displays ownership tree (parent → company → subsidiaries); each entry shows data source badge, last-refreshed date, staleness notice if stale; SABO group links where present; "No data found" state with EDGAR/OpenCorporates manual links
- [ ] T096 [P] [US6] Create frontend/src/routes/compass/+page.svelte — Conglomerate Compass page; public (no auth); renders CompassSearch; page title and brief explainer; mobile-first layout
- [ ] T097 [P] [US6] Create frontend/src/lib/components/ConstellationPanel.svelte — on parent group page: shows constellation panel with subsidiary group cards (name, member count, total score independently); on subsidiary group page: shows parent group link + "Available at [Parent] level" tactic section per FR-047
- [ ] T098 [US6] Update frontend/src/routes/proposals/new/+page.svelte — after POST /proposals returns constellation_offer, render a constellation attachment prompt ("We detected [Company] is a subsidiary of [Parent] — attach to existing [Parent] group?"); user can accept (resubmits with constellation_parent_group_id) or skip

**Checkpoint**: Quickstart Scenario 5 passes. Compass returns EDGAR data with SABO group links. Constellation panel on group pages shows labeled rollup. Proposal flow surfaces subsidiary detection.

---

## Phase 9: User Story 7 — Company Acknowledgment Status (Priority: P7)

**Goal**: Every group page shows the acknowledgment tracker. Staff can update it. Updates appear in real-time (≤60 seconds). BehaviorChanged is the visible win state.

**Independent Test**: Quickstart Scenario 6 — non-staff 403; valid transitions succeed; acknowledgment_update SSE event fires within 60 seconds; invalid transition returns 422.

### API Endpoints (US7)

- [ ] T099 [US7] Add PATCH /api/v1/groups/{slug}/acknowledgment to backend/src/api/v1/groups.py — is_staff required; validates state transition (NoResponse→StatementIssued, NoResponse→BehaviorChanged, StatementIssued→BehaviorChanged; all others 422); updates AcknowledgmentRecord; publishes acknowledgment_update SSE event via feed_service; if BehaviorChanged, also publishes win_state event and triggers notification to opted-in group members (Proton Mail SMTP, async via Celery)

### Frontend (US7)

- [ ] T100 [US7] Update frontend/src/lib/stores/feed.ts — handle acknowledgment_update SSE event; update group acknowledgment state in store; handle milestone_crossed event; update milestone display
- [ ] T101 [P] [US7] Update frontend/src/lib/components/AcknowledgmentTracker.svelte — reactive to feed store acknowledgment_update events; BehaviorChanged state: full-width win banner with celebratory styling, source link, campaign duration and total action count; StatementIssued state: status badge + source link; NoResponse state: days-since-launch counter
- [ ] T102 [P] [US7] Update frontend/src/lib/components/MilestoneTracker.svelte — reactive to milestone_crossed feed events; crossed milestones show celebration badge + crossed_at timestamp; uncrossed milestones show progress toward threshold

**Checkpoint**: Quickstart Scenario 6 passes. Acknowledgment tracker live-updates. Win state visually prominent.

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Deployment readiness, security hardening, pre-launch gates.

- [ ] T103 Add FastAPI rate limiting middleware to backend/main.py — 5 failed login attempts per 15 min per IP on POST /auth/login; 429 response with retry-after header
- [ ] T104 [P] Implement consistent error response format across all backend API endpoints per contracts/api-v1.md — {error: {code, message, detail}}; add exception handlers to backend/main.py for HTTPException, ValidationError, IntegrityError
- [ ] T105 [P] Add Plausible analytics script to frontend/src/app.html — self-hosted Plausible snippet; no Google Analytics; respects user's Do Not Track header
- [ ] T106 [P] Configure Cloudflare CDN cache headers for frontend static assets — far-future Cache-Control on immutable build output; no-cache on HTML pages (SSR); in nginx/sabo.conf
- [ ] T107 Conduct mobile usability pass — load all routes at 375px viewport; verify no horizontal scrolling; test group page, tactic library, compass page, proposal form, profile page; fix any layout breaks
- [ ] T108 [P] Conduct partisan-language audit — manually review all four seeded grievance bodies and tactic descriptions; confirm zero partisan framing, ideological signaling, or editorial opinion (SC-006 gate)
- [ ] T109 Verify primary source links for all four seeded campaigns — confirm all grievance.primary_source_url links resolve to live government/court/regulatory records; cache snapshot if any is at risk of moving
- [ ] T110 Run Quickstart Scenario 1 end-to-end for all four seeded campaigns (SC-009 gate) — Amazon, Meta, Cigna, Wells Fargo must each pass all 10 steps; document results
- [ ] T111 [P] Add CORS configuration to backend/main.py — allow frontend origin only; credentials=True for cookie auth; no wildcard origins in production
- [ ] T112 Create backend/src/scripts/run_migrations.sh — wrapper script that runs `alembic upgrade head` for deployment automation
- [ ] T113 [P] Create deployment checklist in docs/deploy.md — environment variables, Docker Compose production overrides, Nginx setup, Cloudflare DNS configuration, Celery worker startup, seed data run instructions

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 — blocks all user stories
- **Phase 3 (US1)**: Depends on Phase 2 — this is the MVP gate; validate before proceeding
- **Phase 4 (US2)**: Depends on Phase 3 (tactic models and endpoints must exist)
- **Phase 5 (US3)**: Depends on Phase 3 (Action model and scoring must exist); can run in parallel with Phase 4
- **Phase 6 (US4)**: Depends on Phase 5 (leaderboard extends the scoring + feed infrastructure)
- **Phase 7 (US5)**: Depends on Phase 2 (Group/Company models); can start in parallel with Phases 4–6
- **Phase 8 (US6)**: Depends on Phase 7 (constellation wires into proposal flow)
- **Phase 9 (US7)**: Depends on Phase 5 (acknowledgment_update is an SSE event)
- **Phase 10 (Polish)**: Depends on all story phases complete

### User Story Dependencies

- **US1 (P1)**: Foundational complete → implement; VALIDATE before proceeding
- **US2 (P2)**: US1 complete (tactic infrastructure exists)
- **US3 (P3)**: US1 complete (Action model exists); parallel with US2
- **US4 (P4)**: US3 complete (leaderboard_update extends SSE infrastructure)
- **US5 (P5)**: Foundational complete (Group/Company models exist); largely independent
- **US6 (P6)**: US5 complete (constellation integrates into proposal flow)
- **US7 (P7)**: US3 complete (acknowledgment_update uses SSE infrastructure)

### Within Each User Story

- Models before services
- Services before API endpoints
- API endpoints before frontend components
- Seed data (T043) runs after all models are migrated

### Parallel Opportunities

- T003, T004, T007, T008 — setup config files (different files)
- T016, T017 — User and Company models (different files)
- T027–T036 — all US1 models (different files, no dependencies on each other)
- T044, T045, T046 — frontend API wrappers (different files)
- T050, T051, T052, T053, T054, T055 — frontend components (different files)

---

## Parallel Execution Examples

### Phase 3 (US1) — Models (can all run simultaneously)
```
T027 backend/src/models/group.py
T028 backend/src/models/grievance.py
T029 backend/src/models/tactic.py
T030 backend/src/models/tactic_group_link.py
T031 backend/src/models/action.py
T032 backend/src/models/proof_artifact.py
T033 backend/src/models/group_membership.py
T034 backend/src/models/user_group_score.py
T035 backend/src/models/milestone.py
T036 backend/src/models/acknowledgment_record.py
```

### Phase 3 (US1) — Frontend components (can all run simultaneously)
```
T050 GrievancePanel.svelte
T051 TacticCard.svelte
T052 TacticLibrary.svelte
T053 ProofUpload.svelte
T054 AcknowledgmentTracker.svelte
T055 MilestoneTracker.svelte
```

### Phases 4–7 — After US1 validates (can run in parallel if team size allows)
```
Phase 4 (US2): T056–T061
Phase 5 (US3): T062–T071
Phase 7 (US5): T078–T083
```

---

## Implementation Strategy

### MVP First (US1 Only — Phases 1–3)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL — blocks all stories)
3. Complete Phase 3: US1 Core Loop
4. **STOP AND VALIDATE**: Run Quickstart Scenario 1 for all four seeded campaigns
5. Deploy internal demo — if the loop works, the platform concept is proven

### Incremental Delivery

1. Setup + Foundational → auth works, DB migrated, seed data loadable
2. US1 → Core loop completable → **MVP gate** → internal launch
3. US2 → Full tactic library experience → library feels complete
4. US3 → Real-time feed → platform feels alive
5. US4 → Leaderboard → social competition layer
6. US5 → Group proposals → community growth unlocked
7. US6 → Conglomerate Compass → ownership transparency tool live
8. US7 → Acknowledgment tracker win state → campaign closure visible
9. Polish → Production hardening → public launch

---

## Notes

- All real-time updates (feed, scores, leaderboard, acknowledgment) must meet the 60-second staleness ceiling (FR-040, SC-004)
- Venting posts must NEVER appear in action feed API responses or SSE streams (FR-018)
- Attribution anonymization is default — only surface display_name if BOTH GroupMembership.public_attribution_opt_in AND User.display_name are set (FR-019, FR-031)
- The constellation rollup on parent group pages must display two separately labeled components — direct actions and subsidiary rollup — never merged (FR-015c)
- SEC EDGAR requests must include a User-Agent header with contact info (EDGAR policy) — set in config as EDGAR_USER_AGENT
- OpenCorporates integration is intentionally absent from all tasks — add only after constitution amendment is ratified
- Seed grievance content (T043) must pass the partisan-language audit (T108) before launch
