# Quickstart & Integration Scenarios: SABO MVP Web Platform

**Branch**: `002-mvp-web-platform` | **Date**: 2026-05-19

These are the canonical integration test scenarios. Each scenario is independently verifiable and maps to one or more user stories from the spec.

---

## Scenario 1: Complete the Core Loop (US1 — P1 gate)

This is the MVP readiness gate. If this scenario fails end-to-end, nothing ships.

**Setup**: Amazon seeded campaign is Active with at least one tactic (routing_level=Subsidiary, impact_category=Revenue).

**Steps**:
1. `GET /api/v1/groups/amazon` → 200, `status=Active`, grievance has `primary_source_url`, `member_count >= 0`, `acknowledgment.status = NoResponse`
2. `GET /api/v1/groups/amazon` (unauthenticated) → 200, same response — no auth required to view
3. `POST /api/v1/auth/register` with `{email, password}` → 201, no session yet
4. `POST /api/v1/auth/login` → 200, `sabo_token` cookie set
5. `POST /api/v1/groups/amazon/join` → 200, `{group_slug: "amazon"}`
6. `GET /api/v1/groups/amazon/tactics?routing_level=Subsidiary&category=Revenue` → 200, at least one tactic returned
7. `POST /api/v1/actions` with `{tactic_id, group_id: amazon_group_id, attestation_type: "Screenshot"}` + `proof_image` file → 201, `{score: N, status: "Submitted"}`
8. Open SSE connection `GET /api/v1/groups/amazon/feed/stream` — verify `new_action` event arrives within 60 seconds
9. `GET /api/v1/groups/amazon/feed` → 200, action appears in feed with anonymized attribution (not user's email)
10. `GET /api/v1/users/me` → `total_score = N`, `score_by_category.Revenue = N`

**Pass criteria**: All 10 steps succeed. Score appears within 60 seconds of step 7 (step 8 verifies). User's email is never exposed in feed response.

---

## Scenario 2: Tactic Library Filtering and Routing (US2)

**Setup**: Library seeded with ≥10 tactics across ≥3 categories; at least one `routing_level=Parent` tactic exists.

**Steps**:
1. `GET /api/v1/tactics?category=Revenue` → 200, all items have `impact_category=Revenue`
2. `GET /api/v1/tactics?category=Regulatory` → 200, all items have `impact_category=Regulatory`
3. `GET /api/v1/tactics/{regulatory_tactic_id}` → 200, `description` contains external portal link + confirmation number submission field reference
4. `GET /api/v1/tactics?routing_level=Parent` → 200, all items have `routing_level=Parent`
5. `GET /api/v1/groups/amazon/tactics` → 200, response has `subsidiary_tactics` array and `parent_level_tactics=null` (standalone group)
6. `POST /api/v1/actions` with a Parent-routing tactic against a Subsidiary group → 400, `ROUTING_LEVEL_MISMATCH` error

**Pass criteria**: Category filter returns only matching tactics. Routing validation rejects mismatched action logging.

---

## Scenario 3: Real-Time Feed and Leaderboard (US3, US4)

**Setup**: Amazon group Active, at least 2 users are members with `leaderboard_opt_in=true` and `public_attribution_opt_in=true`.

**Steps**:
1. Open two SSE connections to `GET /api/v1/groups/amazon/feed/stream` (simulate two browser clients)
2. User A logs an action via `POST /api/v1/actions`
3. Verify both SSE connections receive `new_action` event within 60 seconds
4. Verify `score_update` event arrives on both connections with updated category total
5. `GET /api/v1/groups/amazon/leaderboard` → 200, User A appears with updated score
6. `PATCH /api/v1/groups/amazon/membership` with `{leaderboard_opt_in: false}` for User A
7. `GET /api/v1/groups/amazon/leaderboard` → User A no longer appears

**Pass criteria**: Both SSE clients receive updates within 60 seconds. Leaderboard opt-out is reflected immediately.

---

## Scenario 4: Group Proposal → Constellation Detection → Auto-Activation (US5)

**Setup**: Amazon group Active. A new company "Whole Foods Market" is being proposed. EDGAR data has Whole Foods as a subsidiary of Amazon.

**Steps**:
1. `POST /api/v1/proposals` with `{company_name: "Whole Foods Market", primary_source_type: "RegulatoryAction", primary_source_url: "https://...", grievance_description: "..."}`
2. Response includes `constellation_offer: {detected_parent: "Amazon.com, Inc.", existing_parent_group: {slug: "amazon"}}`
3. Resubmit `POST /api/v1/proposals` with `constellation_parent_group_id` set to Amazon group's id → 201, `status=Pending`
4. `POST /api/v1/proposals/{id}/upvote` from 50 different authenticated users (or test with threshold set to 2 in test environment)
5. After final upvote, proposal `status=Approved` and `resulting_group_id` is populated
6. `GET /api/v1/groups/whole-foods` → 200, `constellation_role=Subsidiary`
7. `GET /api/v1/groups/amazon` → `constellation.subsidiaries` includes Whole Foods
8. `GET /api/v1/groups/amazon` scores show `subsidiary_rollup` with Whole Foods listed separately

**Pass criteria**: Constellation offer surfaces on detection. Auto-activation occurs at threshold. Parent scores show labeled subsidiary rollup, not merged totals.

---

## Scenario 5: Conglomerate Compass Search (US6)

**Setup**: Amazon OwnershipRecord cached from EDGAR. Whole Foods subsidiary listed.

**Steps**:
1. `GET /api/v1/compass/search?q=Amazon` → 200, results include Amazon with `subsidiaries` array containing Whole Foods
2. Result includes `sabo_group: {slug: "amazon"}` linking to the active SABO group
3. `GET /api/v1/compass/search?q=Whole+Foods` → 200, result shows `parent: {name: "Amazon.com, Inc."}`, `sabo_group: {slug: "whole-foods"}` if group is active
4. `GET /api/v1/compass/search?q=UnknownCorp` → 200, `results: []` or result with no subsidiaries/parent + no sabo_group
5. Staff force-refresh: `POST /api/v1/admin/ownership-records/{company_id}/refresh` → 202, task queued
6. After task completes, `GET /api/v1/compass/companies/{cik}` → `last_refreshed` updated

**Pass criteria**: Search returns accurate ownership data with SABO group links where applicable. Unknown companies return gracefully. Force-refresh is async and non-blocking.

---

## Scenario 6: Acknowledgment Tracker Win State (US7)

**Setup**: Cigna group Active with `acknowledgment.status=NoResponse`.

**Steps**:
1. `PATCH /api/v1/groups/cigna/acknowledgment` as non-staff user → 403 Forbidden
2. `PATCH /api/v1/groups/cigna/acknowledgment` as staff with `{status: "StatementIssued", source_url: "https://...", source_label: "Cigna Statement, May 2026"}` → 200
3. SSE connection to `GET /api/v1/groups/cigna/feed/stream` receives `acknowledgment_update` event within 60 seconds
4. `GET /api/v1/groups/cigna` → `acknowledgment.status = "StatementIssued"`, `source_url` populated
5. `PATCH /api/v1/groups/cigna/acknowledgment` with `{status: "BehaviorChanged", ...}` → 200
6. `GET /api/v1/groups/cigna` → `acknowledgment.status = "BehaviorChanged"`
7. Attempt invalid transition: `PATCH` with `{status: "NoResponse"}` → 422 invalid transition

**Pass criteria**: Only staff can update tracker. Update is reflected in SSE stream within 60 seconds. Invalid state transitions are rejected.

---

## Scenario 7: Privacy — Anonymization by Default

**Setup**: User has `public_attribution_opt_in=false` at group level.

**Steps**:
1. User logs an action with `is_public=false`
2. `GET /api/v1/groups/amazon/feed` → action attribution shows anonymized label (e.g., "A member"), never the user's email or display_name
3. `GET /api/v1/groups/amazon/leaderboard` → user does not appear
4. `GET /api/v1/users/me` → full score visible to the authenticated user themselves
5. User enables `public_attribution_opt_in=true` via `PATCH /api/v1/groups/amazon/membership`
6. Future actions appear with `display_name` in the feed; past actions remain anonymized (retrospective exposure is not applied)

**Pass criteria**: Anonymization is the default. Opt-in is per-group and forward-only for past actions.

---

## Scenario 8: Venting Feed Separation

**Setup**: Group Active; venting surface exists.

**Steps**:
1. `POST /api/v1/groups/amazon/venting` with post body → 201
2. `GET /api/v1/groups/amazon/feed` (action board) → venting post does NOT appear
3. `GET /api/v1/groups/amazon/venting` → venting post appears
4. SSE `GET /api/v1/groups/amazon/feed/stream` → `new_action` events fire for actions only, never for venting posts

**Pass criteria**: Venting and action board are strictly separate; no cross-contamination.

---

## Scenario 9: Duplicate Action Prevention

**Steps**:
1. User logs "Cancel Amazon Prime" tactic against Amazon group → 201
2. User attempts to log same tactic against Amazon group again → 409, `DUPLICATE_ACTION` error with warning message
3. `GET /api/v1/admin/flags` as staff → the flagged second-attempt record is visible for review

**Pass criteria**: Duplicate is rejected at submission time. A review flag is created for staff.

---

## Scenario 10: Proof Artifact Upload and Display

**Steps**:
1. `POST /api/v1/actions` with multipart form including `proof_image` (JPEG, < 10 MB) → 201, `proof_thumbnail_url` populated
2. `GET /api/v1/groups/amazon/feed` → action entry has non-null `proof_thumbnail_url`
3. Thumbnail URL resolves to a Hetzner Object Storage-backed Cloudflare CDN URL
4. `POST /api/v1/actions` with no proof artifact and no confirmation text → 422, `PROOF_REQUIRED` error

**Pass criteria**: Images are stored in Hetzner Object Storage and served via CDN. Actions without proof are rejected.
