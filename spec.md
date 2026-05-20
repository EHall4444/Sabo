# Feature Specification: SABO MVP Web Platform

**Feature Branch**: `002-mvp-web-platform`

**Created**: 2026-05-19

**Status**: Draft

**Input**: User description: "Build SABO — a web-based consumer activism platform where users join groups organized around specific corporate grievances and coordinate financial and reputational pressure campaigns against those companies."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Complete the Core Loop on a Seeded Campaign (Priority: P1)

A user lands on SABO, reads the documented grievance for one of the four seeded companies, joins the group, picks a tactic from the library, takes action, reports back with proof, and sees their contribution reflected in the group's live aggregate score.

**Why this priority**: The core loop is the product. If a user cannot complete it start-to-finish for at least one seeded campaign, nothing else matters. This is the gate that determines MVP readiness.

**Independent Test**: A single user with a new account can: view a seeded campaign grievance, join the group, select and execute a tactic, submit proof of action, and see the group's aggregate score and their personal contribution score update — without any assistance or documentation.

**Acceptance Scenarios**:

1. **Given** a visitor lands on the Amazon group page without an account, **When** they scroll the page, **Then** they see: the documented grievance with a clickable primary source link, the active member count, the live tactic feed, the milestone tracker, and the company acknowledgment status — all without logging in.
2. **Given** a visitor views the Amazon group page, **When** they click the primary source link, **Then** they are taken directly to the government filing, court judgment, or regulatory enforcement action that documents the grievance.
3. **Given** an unauthenticated visitor, **When** they attempt to join a group or log an action, **Then** they are prompted to create an account or log in; account creation takes under 2 minutes with only an email and password.
4. **Given** a logged-in user on the Amazon group page, **When** they open the Tactic Library and select "Cancel Amazon Prime subscription," **Then** they see effort level (low/medium/high), estimated impact category (Revenue), usage count, and community rating before committing.
5. **Given** a logged-in user who completed a tactic, **When** they report back by uploading a screenshot of their cancellation confirmation, **Then** their action appears in the group's action feed and their personal contribution score increments by the tactic's weighted value.
6. **Given** a logged-in user who reported an action, **When** they view the group's action feed, **Then** their post appears anonymized by default (e.g., "A member canceled their subscription") — individual attribution only if they opted in.

---

### User Story 2 - Browse and Execute Tactics from the Library (Priority: P2)

A logged-in user can open the Tactic Library, filter by impact category, read the detail card for any tactic (effort, estimated impact, usage count, community rating, step-by-step instructions), and link a tactic execution to a specific group action.

**Why this priority**: The Tactic Library is the action engine of the platform. Without a well-organized, filterable library, users face a blank page when asked to act. This is the second most critical surface after the group page itself.

**Independent Test**: The library loads with 10–15 tactics pre-populated, filterable by at least three of the five impact categories. A user can select a tactic, read its full detail card, execute or log it against a campaign, and the library usage count for that tactic increments.

**Acceptance Scenarios**:

1. **Given** a user opens the Tactic Library, **When** they apply a "Revenue" filter, **Then** only Revenue-category tactics are displayed and the count reflects the filter.
2. **Given** a user views a tactic card, **When** they read it, **Then** they see: impact category badge, effort level (Low/Medium/High), estimated corporate cost rating (1–5), usage count (how many times this tactic has been executed across all groups), community rating (1–5 stars), and step-by-step instructions.
3. **Given** a user selects a tactic and clicks "Take Action," **When** they are prompted, **Then** they can associate it with any group they have joined.
4. **Given** a user selects a "Regulatory" tactic (e.g., "File a CFPB complaint"), **When** they read the instructions, **Then** the instructions guide them to the external government portal, explain what to submit, and provide a form to paste their confirmation number as proof of completion.
5. **Given** a third-party developer submits a tactic, **When** it is submitted, **Then** they are required to sign the contributor agreement before the tactic enters the review queue.
6. **Given** a submitted tactic is reviewed by platform staff, **When** it is approved, **Then** it appears in the library tagged with the contributor name, impact category, and a "Legal Review: Passed" indicator.

---

### User Story 3 - Follow the Real-Time Action Feed (Priority: P3)

A user viewing a group page can see a live action feed showing recent member actions — receipts, screenshots, and confirmation posts — with the tactic used and the contribution score earned per post. The feed updates in near-real-time without a page refresh.

**Why this priority**: The action feed is what makes the platform feel alive. A static scoreboard is a database. A live feed of humans doing things is a movement. Social proof sustains participation after the initial action.

**Independent Test**: The action feed on any group page updates within 60 seconds when a new action is submitted. Each entry shows: anonymized member attribution, tactic name, contribution score, and the attached proof artifact. The venting surface (if present) is visually distinct from the action board.

**Acceptance Scenarios**:

1. **Given** a user views the group action feed, **When** a new action is submitted by any member, **Then** it appears in the feed within 60 seconds without a manual page refresh.
2. **Given** a user posts a receipt (screenshot of a cancellation email), **When** the post is submitted, **Then** other feed viewers see it as: "[Anonymized member] canceled their subscription — Revenue tactic — +12 points."
3. **Given** a user who opted into public attribution posts an action, **When** other users view the feed, **Then** the post shows their display name instead of the anonymized label.
4. **Given** the action feed is viewed, **When** a venting post exists, **Then** it appears in a visually separated "Community" surface — not on the action board — and is never promoted above verified action posts.
5. **Given** a group has received no actions in 7 days, **When** a user views the feed, **Then** a prompt encourages the first action of the week rather than showing a dead feed.
6. **Given** a proof artifact (screenshot) is attached to a post, **When** the post appears in the feed, **Then** the thumbnail is visible inline; clicking opens the full image.

---

### User Story 4 - Track Contribution Score and Group Leaderboard (Priority: P4)

A logged-in user can view their personal contribution score (weighted by tactic impact), see their score broken down by group and tactic category, and opt into the group leaderboard to see their ranking relative to other members.

**Why this priority**: Contribution scoring closes the social reward stage of the core loop. Users need to feel their individual action added measurable weight to the campaign — and optionally, to see how they rank against others who share the same target.

**Independent Test**: After logging two actions of different tactic categories, a user's profile page shows a score breakdown by category. Opting into the leaderboard places them in the visible ranking; opting out removes them. Scores update within 60 seconds.

**Acceptance Scenarios**:

1. **Given** a user logs a Revenue-category tactic and a Regulatory-category tactic, **When** they view their profile, **Then** their score shows a breakdown: Revenue: X pts, Regulatory: Y pts, Total: X+Y pts.
2. **Given** a tactic is rated higher impact than another, **When** both are executed and scored, **Then** the higher-impact tactic awards proportionally more points.
3. **Given** a user opts into the group leaderboard, **When** the leaderboard is displayed, **Then** their display name and score appear in ranked order relative to other opted-in members.
4. **Given** a user opts out of the leaderboard, **When** the leaderboard is displayed, **Then** their score is not visible to any other user.
5. **Given** a leaderboard is viewed, **When** a new action is submitted by a ranked member, **Then** their position and score update within 60 seconds.
6. **Given** a user views their profile, **When** they browse their action history, **Then** each past action shows: group name, tactic name, proof artifact, score earned, and submission date.

---

### User Story 5 - Propose and Validate a New Group (Priority: P5)

A logged-in user can propose a new corporate grievance group by submitting a company name, a documented grievance description, and a link to a primary source document. The proposal enters a community review phase where other members upvote it; once it crosses an upvote threshold, the group goes live.

**Why this priority**: The four seeded campaigns are a launch vehicle, not a ceiling. The platform scales through user-proposed campaigns. Grievance validation ensures the quality and factual grounding required by the Bipartisan by Design principle — no editorial opinion, no unverifiable claims.

**Independent Test**: A user can submit a group proposal with a company name, grievance description, and primary source link. The proposal appears in a public review queue. Other users can upvote it. When the upvote threshold is reached, the group page goes live and is joinable.

**Acceptance Scenarios**:

1. **Given** a user submits a new group proposal, **When** they fill out the form, **Then** they are required to provide: target company name, a factual grievance description, and a URL to a verifiable primary source document (government filing, court judgment, regulatory action).
2. **Given** a proposal is submitted with an editorial blog post as the primary source, **When** it is reviewed, **Then** it is rejected with a clear explanation that only government filings, court judgments, and regulatory enforcement actions qualify.
3. **Given** a valid proposal is submitted, **When** it enters the community review queue, **Then** any logged-in user can read it and cast one upvote (no duplicate votes).
4. **Given** a proposal reaches the upvote threshold (configurable, default 50 upvotes), **When** the threshold is crossed, **Then** the group page is automatically created, the proposer is notified, and the group appears in the main campaign directory.
5. **Given** a group page goes live, **When** any user reads the grievance, **Then** the primary source link is visible and verified to be a non-editorial source.
6. **Given** a group proposal is rejected, **When** the proposer is notified, **Then** they receive the specific reason and are allowed to resubmit with a corrected primary source.

---

### User Story 6 - Explore Corporate Ownership with the Conglomerate Compass (Priority: P6)

Any visitor — logged in or not — can open the Conglomerate Compass, search for a company by name, and see its known corporate ownership structure: who owns it, what it owns, and whether any part of that structure is the subject of an active SABO group or constellation.

**Why this priority**: Transparency about who really owns a company is itself an act of counter-power. Many users don't know that filing a complaint against "Evernorth" is pressure on Cigna, or that their complaint against Instagram reaches Meta. The Conglomerate Compass makes the corporate structure visible and connects it directly to SABO action opportunities.

**Independent Test**: A user can search for any of the four seeded campaign companies (or a known subsidiary), see the ownership tree sourced from OpenCorporates/SEC EDGAR, and find a direct link to the corresponding SABO group. No account required.

**Acceptance Scenarios**:

1. **Given** a visitor searches for "Whole Foods" in the Conglomerate Compass, **When** results load, **Then** they see Whole Foods listed as a subsidiary of Amazon, with a link to the Amazon SABO group.
2. **Given** a visitor searches for "Instagram," **When** results load, **Then** they see Instagram listed as a subsidiary of Meta, with a link to the Meta SABO group.
3. **Given** a visitor searches for a company with no active SABO group, **When** results load, **Then** they see the ownership structure and a prompt to propose a new group if a grievance exists.
4. **Given** ownership data for a searched company was last refreshed over 30 days ago, **When** results are displayed, **Then** a visible staleness notice shows the last-refreshed date and informs the user that data may be outdated.
5. **Given** OpenCorporates and EDGAR return no ownership data for a searched company, **When** results load, **Then** the Compass displays "No ownership data found" and links to EDGAR and OpenCorporates for manual lookup.
6. **Given** any Compass result, **When** the user views it, **Then** the data source (OpenCorporates, SEC EDGAR Exhibit 21, or both) is clearly labeled alongside each ownership relationship shown.

---

### User Story 7 - Monitor Company Acknowledgment Status (Priority: P7)

A user visiting any group page can see the company acknowledgment tracker — a visible status indicator showing whether the target company has: not yet responded, issued a statement, or changed behavior in response to campaign pressure. This is the visible win state of the platform.

**Why this priority**: Closing the loop with a corporate response is what distinguishes SABO from a petition site. Users need to see that pressure works. The acknowledgment tracker is the "did it work?" answer displayed on the campaign page for every visitor.

**Independent Test**: Every group page has an acknowledgment tracker in one of three states: No Response, Statement Issued, or Behavior Changed. Platform staff can update the state with a source link. The update appears on the group page within 60 seconds and is visible to all visitors including anonymous ones.

**Acceptance Scenarios**:

1. **Given** a group page is viewed before any corporate response, **When** the tracker is displayed, **Then** it shows "No Response" with the campaign start date and days since launch.
2. **Given** a target company issues a public statement, **When** a platform staff member logs the response with a source link, **Then** the tracker updates to "Statement Issued," the source link is displayed, and the date is recorded.
3. **Given** the tracker updates to "Statement Issued," **When** any member views the group page, **Then** a celebratory notification appears highlighting the corporate response and crediting the campaign.
4. **Given** a target company makes a documented policy change or settlement in response to campaign pressure, **When** a platform staff member logs it with a source link, **Then** the tracker updates to "Behavior Changed" — the ultimate win state — and a full-width milestone banner appears on the group page.
5. **Given** the tracker is in any state, **When** the source link is clicked, **Then** users are taken to the verifiable external record (press release, regulatory filing, news article) — no editorial interpretation.
6. **Given** a "Behavior Changed" win is logged, **When** all group members who are opted into notifications are notified, **Then** the notification credits the collective action count and the duration of the campaign.

---

### Edge Cases

- A user tries to log the same tactic twice against the same campaign: the system warns them and does not double-count the score; the second submission is flagged for review.
- A primary source URL on a group grievance goes dead: the platform must cache a snapshot or flag the link for staff review; the group is not removed for a broken link alone.
- A tactic is found to be illegal after approval and publication: platform staff can immediately remove it; members who logged it are notified and their action is voided from the score.
- A company being targeted creates a SABO account and upvotes a competing group proposal to dilute attention: the upvote mechanism does not expose voter identity to targets; staff can audit suspicious upvote patterns.
- A user submits a fraudulent proof (fabricated screenshot): MVP uses trust-based attestation; fraud reports go to a staff review queue; confirmed fraud voids the action and warns the account.
- A seeded campaign's primary source link moves or goes behind a paywall: platform maintains a cached reference; staff must update the link within 7 days of detection.
- A proposal is submitted for a company that is already the subject of an active group: the system detects the duplicate by company name match and prompts the proposer to join the existing group instead.

## Clarifications

### Session 2026-05-19

- Q: What depth does the constellation hierarchy support? → A: Two levels for MVP (one parent group, one or more direct subsidiary groups). Data model must be extensible for deeper nesting in a future phase but multi-level traversal is not required at launch.
- Q: When a user executes a tactic, what determines which group receives the logged action? → A: Tactic level determines the destination. Subsidiary-level tactics (regulatory complaints, product reviews, etc.) log to the subsidiary group. Parent-level tactics (shareholder resolutions, board outreach, etc.) log to the parent group. Parent aggregate = parent's own direct actions + sum of all subsidiary group actions. No user choice is required at reporting time.
- Q: How frequently is corporate ownership data refreshed from OpenCorporates/SEC EDGAR? → A: Fetched once at group creation time and cached. Cache expires after 30 days. Staff can force-refresh any group's ownership data on demand. No continuous background sync required for MVP.
- Q: How do parent-level tactics appear on a subsidiary group page, and should there be a standalone ownership lookup tool? → A: Parent-level tactics appear in a visually distinct labeled section ("Available at the [Parent Company] level") with a direct link to the parent group page — not hidden, not greyed out. Additionally, the platform will include the **Conglomerate Compass** — a standalone, publicly accessible tool for looking up corporate ownership hierarchies (who owns whom). Powered by the same OpenCorporates/SEC EDGAR data used for constellation detection.
- Q: Which ownership data source is used for MVP — SEC EDGAR, OpenCorporates, or both? → A: SEC EDGAR Exhibit 21 is the primary source for MVP (free, no vendor approval required, covers all US public companies including all four seeded campaigns). OpenCorporates is the desired long-term primary source for broader global/private company coverage; it MUST be added to the approved vendor stack via constitution amendment as soon as ethically feasible after launch. The Conglomerate Compass and constellation detection are architected to support OpenCorporates as a drop-in addition when approval is granted.

## Requirements *(mandatory)*

### Functional Requirements

**Group Pages**

- **FR-001**: Each group page MUST display: target company name and logo, the documented grievance with a primary source link, active member count, live tactic feed, milestone tracker, and company acknowledgment status.
- **FR-002**: All group pages MUST be publicly viewable without an account.
- **FR-003**: The grievance MUST be anchored to a verifiable primary source document: government filing, court judgment, or regulatory enforcement action. Editorial opinion does not qualify.
- **FR-004**: Group pages MUST display aggregate pressure scores broken down by tactic impact category (Reputation, Revenue, Regulatory, Capital, Operations).
- **FR-005**: Milestone thresholds MUST be configurable per group. When a threshold is crossed, a milestone celebration MUST appear on the group page.
- **FR-006**: The company acknowledgment tracker MUST be visible on every group page, in one of three states: No Response, Statement Issued, Behavior Changed.
- **FR-007**: Platform staff MUST be able to update the acknowledgment tracker with a source link; updates MUST appear within 60 seconds.
- **FR-008**: All group copy MUST use neutral, factual language — no partisan framing, ideological signaling, or editorial opinion.

**Tactic Library**

- **FR-009**: The Tactic Library MUST contain at least 10 tactics at launch, spanning at least three of the five impact categories.
- **FR-010**: Each tactic MUST display: impact category, effort level (Low/Medium/High), estimated corporate cost rating (1–5), usage count, community rating (1–5 stars), and step-by-step instructions.
- **FR-011**: The library MUST be filterable by impact category.
- **FR-012**: Each tactic MUST describe a legal action any individual can take alone.
- **FR-013**: Third-party tactic submissions MUST require a signed contributor agreement before entering the review queue.
- **FR-014**: All submitted tactics MUST pass a legal review before publication.
- **FR-015**: Tactics linked to specific campaigns MUST show campaign-specific customization (e.g., "Cancel Amazon Prime" vs. generic "Cancel Subscription").
- **FR-015a**: Each tactic MUST be designated with a routing level: Subsidiary (logs to subsidiary group) or Parent (logs to parent group). This designation is set at tactic creation and displayed on the tactic card.
- **FR-015b**: An Action MUST be logged to exactly one group, determined by the tactic's routing level. No user selection of destination group is required or offered at reporting time.
- **FR-015c**: A parent group's aggregate score display MUST show two clearly labeled components: (1) actions logged directly to the parent group, and (2) a rollup total of all subsidiary group actions. These MUST NOT be summed into a single unlabeled number.

**Action Feed**

- **FR-016**: The action feed on each group page MUST update within 60 seconds of a new action being submitted, without requiring a page refresh.
- **FR-017**: Each feed entry MUST show: anonymized or opted-in member attribution, tactic name, contribution score earned, and proof artifact thumbnail.
- **FR-018**: The action board MUST display only verified or user-attested contributions. Venting posts MUST appear on a separate surface and MUST NOT be promoted above action posts.
- **FR-019**: Individual attribution MUST be anonymized by default. Opt-in public attribution displays the user's chosen display name.

**Contribution Scoring**

- **FR-020**: Contribution scores MUST be weighted by tactic impact category, not raw action count.
- **FR-021**: Personal score breakdowns MUST be visible on each user's profile page, organized by group and tactic category.
- **FR-022**: Each group MUST have an opt-in leaderboard displaying ranked scores for members who have enabled public attribution.
- **FR-023**: Leaderboard scores MUST update within 60 seconds of a new action being scored.
- **FR-024**: Opting out of the leaderboard MUST make the user's score invisible to all other users immediately.

**Constellation & Ownership**

- **FR-041**: When a user submits a new group proposal, the platform MUST query SEC EDGAR Exhibit 21 filings to determine whether the target company is a known subsidiary of a larger corporate entity. The integration layer MUST be designed so that OpenCorporates can be added as a data source without a structural change once vendor approval is granted.
- **FR-042**: If the target company is detected as a subsidiary, the platform MUST offer the proposer the option to attach the new group to the existing or proposed parent group as a constellation member. This offer MUST be skippable.
- **FR-043**: Corporate ownership data retrieved at group creation MUST be cached with a 30-day TTL. Staff MUST be able to force-refresh ownership data for any group without a code deploy.
- **FR-044**: If ownership APIs are unavailable at group creation time, the proposal MUST proceed without constellation detection. Staff can manually attach constellation membership after the group goes live.
- **FR-045**: A parent group page MUST display its constellation — the list of connected subsidiary groups — with each subsidiary's name, member count, and aggregate pressure score shown independently.
- **FR-046**: A subsidiary group page MUST display a visible link to its parent group. The parent group's total rollup metrics MUST NOT appear on the subsidiary page to avoid inflating the subsidiary's apparent impact.
- **FR-047**: On a subsidiary group page, parent-level tactics (e.g., shareholder resolutions, board outreach) MUST appear in a visually distinct, labeled section — "Available at the [Parent Company] level" — with a direct link to the parent group page. They MUST NOT be hidden or rendered unclickable.
- **FR-048**: The **Conglomerate Compass** MUST be a standalone, publicly accessible feature (no account required) that allows any user to search for a company by name and view its known corporate ownership hierarchy: its parent entity (if any) and its known subsidiaries (if any). For MVP, data is sourced from SEC EDGAR Exhibit 21 filings. The Compass MUST be architected to incorporate OpenCorporates as an additional source — extending coverage to private and non-US companies — once OpenCorporates is approved via constitution amendment. Where a searched company matches an active SABO group or constellation, the Compass MUST surface a direct link to that group.
- **FR-049**: Conglomerate Compass search results MUST display the data source and the last-refreshed date for each ownership record, so users can assess freshness.

**Grievance Validation**

- **FR-025**: New group proposals MUST require: target company name, factual grievance description, and a primary source document link.
- **FR-026**: Proposals sourced to editorial content MUST be rejected with an explanation of qualifying source types.
- **FR-027**: Valid proposals MUST enter a community review queue where any logged-in user can cast one upvote (no duplicates).
- **FR-028**: When a proposal crosses the upvote threshold, the group MUST go live automatically and the proposer MUST be notified.
- **FR-029**: Proposals for companies already active in an existing group MUST surface the duplicate and prompt the user to join the existing group.

**Authentication and Privacy**

- **FR-030**: Account creation MUST require only an email address and password. No real name, phone number, or government ID required.
- **FR-031**: All user actions MUST be attributed to anonymized aggregates by default. Individual attribution is opt-in only.
- **FR-032**: Account creation MUST NOT be required to view group pages, the tactic library, or the action feed.
- **FR-033**: The platform MUST NEVER sell, license, or transfer user data to any third party.
- **FR-034**: Logging MUST be minimal — only what is required to compute scores and detect fraud.

**Seeded Campaigns**

- **FR-035**: The four seeded campaigns (Amazon, Meta, Cigna, Wells Fargo) MUST be live at launch, each pre-populated with a documented grievance sourced to a primary record and a curated set of relevant tactics from the library.

**Technical Quality**

- **FR-036**: All UI MUST be designed mobile-first; layouts MUST be fully usable on mobile viewports without horizontal scrolling.
- **FR-037**: The data layer, API layer, and UI layer MUST be cleanly separated.
- **FR-038**: APIs MUST be versioned and documented before any client consumes them.
- **FR-039**: Database schema changes MUST be managed via migrations, never applied manually.
- **FR-040**: Real-time updates (feed, scores, leaderboard, acknowledgment tracker) MUST have a maximum staleness of 60 seconds.

### Key Entities

- **Group**: A corporate grievance campaign. Has a target company, a grievance (with primary source), member count, pressure score by tactic category, milestone thresholds, and an acknowledgment status. May be seeded (pre-populated) or user-proposed. May optionally be designated as a parent group (has subsidiary groups) or a subsidiary group (has exactly one parent group). For MVP, hierarchy is max two levels deep; the data model MUST support deeper nesting for a future phase.
- **Constellation**: The relationship connecting a parent group to its subsidiary groups. A constellation has one parent group and one or more subsidiary groups. Subsidiary group metrics display independently; parent group metrics display both the parent's own direct-member actions and a clearly labeled rollup of all subsidiary group actions, with no double-counting.
- **ConstellationMembership**: Tracks which groups are connected in a constellation (parent_group_id → subsidiary_group_id). Enforces the two-level MVP constraint.
- **OwnershipRecord**: A cached corporate ownership lookup result for a company. Contains the company name, its parent entity (if any), its known subsidiaries (if any), data source (OpenCorporates / SEC EDGAR / both), and a last-refreshed timestamp. Powers both constellation detection and the Conglomerate Compass. TTL: 30 days. Staff can force-refresh.
- **Grievance**: The documented corporate act anchoring a group. Immutable once published. Requires a verifiable primary source reference (URL, filing ID, case number). Linked 1:1 to a Group.
- **Tactic**: A legal individual action. Has an impact category, effort level, corporate cost rating, usage count, community rating, and step-by-step instructions. May be linked to one or more groups with campaign-specific copy.
- **Action**: A user's execution of a tactic within a group. Contains proof artifact(s) (screenshot, confirmation number, photo), a timestamp, attestation flag, and a computed contribution score. Feeds both the action feed and the scoring system. The destination group is determined by the tactic's designated level: subsidiary-level tactics log to the subsidiary group, parent-level tactics log to the parent group. An action always belongs to exactly one group.
- **User**: A registered account with email and password. Stores: joined groups, tactic history, contribution scores per group and category, public attribution opt-in flag, and leaderboard opt-in flag per group.
- **Milestone**: A configurable aggregate threshold for a group (e.g., "1,000 Revenue actions completed"). Has a celebration state, a crossed-at timestamp, and a visible badge on the group page.
- **AcknowledgmentRecord**: The company response log for a group. Tracks state (No Response / Statement Issued / Behavior Changed), source URL, staff author, and timestamp. Pinned to the group page.
- **GroupProposal**: A user-submitted group request in the community review phase. Has a company name, grievance description, primary source URL, proposer, upvote count, and lifecycle state (Pending / Approved / Rejected).
- **Vote**: A single upvote cast by a logged-in user on a GroupProposal. Enforced one-per-user-per-proposal.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A first-time user can complete the full core loop (land on group page → join group → pick tactic → report action with proof → see score update) in under 10 minutes with no instructions.
- **SC-002**: All four seeded campaigns are live at launch with documented grievances linked to verifiable primary source records and at least 3 pre-loaded tactics each.
- **SC-003**: The Tactic Library contains at least 10 tactics across at least 3 impact categories at launch.
- **SC-004**: The action feed, leaderboard, and acknowledgment tracker all reflect new data within 60 seconds of submission — no manual refresh required.
- **SC-005**: Group pages load fully on a mobile device in under 3 seconds on a standard 4G connection.
- **SC-006**: 100% of group grievance copy passes a partisan-language audit — zero partisan, ideological, or editorial language.
- **SC-007**: No user personally identifiable information is visible to other users without explicit opt-in.
- **SC-008**: A new group proposal can be submitted, reach the upvote threshold, and go live without any staff intervention beyond the initial source-type review.
- **SC-009**: The full core loop is completable end-to-end for all four seeded campaigns before any public launch.

## Assumptions

- MVP is web-only — no mobile app, no browser extension, no desktop client.
- The four seeded campaigns (Amazon, Meta, Cigna, Wells Fargo) are manually curated by SABO staff; automated campaign discovery is a future phase.
- Upvote threshold for new group proposals defaults to 50 upvotes; this is configurable by staff without a code deploy.
- The investment fund structure (501(c)(4)) runs in parallel and does not block platform launch.
- Tactic community ratings (1–5 stars) are submitted by logged-in users who have executed the tactic at least once.
- Proof verification for MVP is trust-based attestation (user self-reports); automated receipt parsing is a future phase.
- Fraud detection is staff-reviewed, not automated, for MVP.
- The platform is hosted on Hetzner with Cloudflare CDN, Plausible analytics — no AWS, Google, or Meta infrastructure at any layer.
- Payments (Stripe) are needed only if a donation or fund participation feature is included; this is not in MVP scope but the vendor is approved.
- Legal counsel reviews tactic contributor agreements and tactic submissions in the Capital and Regulatory categories before those library sections open to third-party contributors.
- A full analytics dashboard is out of scope for MVP; staff uses Plausible for basic traffic analytics.
- The constellation hierarchy is strictly two levels for MVP: one parent group and one or more direct subsidiary groups. Multi-level nesting (e.g., grandparent → parent → subsidiary) is not supported at launch but the data model must not foreclose it.
- Corporate ownership data is sourced from OpenCorporates API and SEC EDGAR Exhibit 21 filings. It is fetched once at group creation, cached for 30 days, and refreshable by staff on demand. API unavailability does not block group creation.
- SEC EDGAR Exhibit 21 is the ownership data source for MVP. It is free, publicly accessible, and requires no vendor approval. All four seeded campaign companies are US-listed public companies covered by EDGAR.
- OpenCorporates is the intended long-term ownership data source (broader global and private company coverage). It is not currently in the approved vendor stack (Constitution v2.1.0, Principle X) and requires a constitution amendment before use. This amendment MUST be pursued as soon as ethically feasible after MVP launch — not treated as an indefinite deferral. The Conglomerate Compass and constellation detection integration layer MUST be designed to accept OpenCorporates as a drop-in addition without structural rework.
