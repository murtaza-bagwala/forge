# Forge

**Forge turns Claude Code from one generic assistant into a team of specialists you can summon on demand.**

Eight workflow skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — all as slash commands.

| Skill | What it does |
|-------|--------------|
| `/forge` | Show all available skills with descriptions and quick-start examples. |
| `/plan-product-review` | Challenge the problem before writing a line of code. Is this the right feature? For whom? |
| `/plan-eng-review` | Lock in architecture, data flow, edge cases, diagrams, and test coverage. |
| `/audit` | Find bugs that survive CI — race conditions, N+1s, trust boundary issues, missing indexes. |
| `/ship` | Sync main, run tests, bump version, commit, push, open PR. |
| `/probe` | QA your feature branch: reads the diff, finds affected pages, tests them automatically. |
| `/design-db` | Design a production database — SQL vs NoSQL decision, ER diagram, full schema, indexes, checklist. |

## Demo: one feature, six modes

The app: a SaaS project management tool. The request: **"add team invitations."**

```
You:   I need to let users invite teammates to their workspace.

You:   /plan-product-review

Claude: "Invite a teammate" sounds simple. Here's what it's
        actually hiding.

        The real job is not sending an email. It's making the first
        five minutes feel effortless — because most invitees never
        complete signup if there's any friction.

        Before building anything:
        → Who can invite? Owner only, or any member?
        → What role does the invitee get? Configurable at send time?
        → What if they already have an account?
        → Do invites expire? Can they be revoked?
        → Is there a seat limit tied to the billing plan?

        The 10-star version:
        - Bulk invite by pasting a list of emails
        - Invitee lands on a pre-filled form (email already in the field)
        - If already a user → one-click join, no signup required
        - Role set at invite time (not after)
        - Seat limit enforced at invite time, not after accept
        - Link expires in 72h, works once, shows clear error if expired
        - Owner sees a pending list with resend / revoke per invite

You:   /plan-eng-review

Claude: Architecture:

          POST /invites  →  pending_invitations row  →  email job
                                                              │
                                                    invitee clicks link
                                                              │
                                               ┌──────────────┴──────────────┐
                                               ▼                             ▼
                                          new user                    existing user
                                     pre-filled signup              one-click join org

        Issue 1: Where should tokens live?
        Recommend B) pending_invitations table (not JWT).
        Why: tokens must be revocable. JWTs can't be revoked without
        a blocklist. A DB row is deletable and auditable.

You:   [implement the plan]

You:   /audit

Claude: CRITICAL — Token never invalidated after use
        app/controllers/invites_controller.rb:47

        Token is validated and membership is created, but
        pending_invitations row is never marked accepted.
        The same invite link works indefinitely — anyone who
        sees the forwarded email can join weeks later.

        Fix: set accepted_at = NOW() before responding.
        Reject if already non-null.

        HIGH — Seat limit race condition
        app/services/invite_service.rb:23

        Seat limit is checked when the invite is SENT, not when
        it's ACCEPTED. 5 invites sent to fill the last 1 seat
        → all 5 could be accepted in the 72h window.

        Fix: re-check inside a DB transaction at accept time.

You:   (fix both issues)

You:   /ship

Claude: Branch: feat/team-invitations — merging origin/main, clean.
        rspec spec/ — 214 examples, 0 failures (4.3s)
        Bumping MICRO: 2.1.4 → 2.1.5
        [commit] [push] [gh pr create]
        PR #312 open → github.com/acme/app/pull/312

You:   /probe

Claude: Diff-aware: 11 files changed.
        Affected routes: /invites, /settings/members, /api/invites

        ✓ Pending invites table renders with resend + revoke
        ✓ Revoke immediately invalidates the token
        ✓ Valid token → pre-filled signup form
        ✓ Existing user → one-click join
        ✗ Expired token → 500 instead of "link expired" page
          GET /invites/abc123 → NoMethodError: pending_invitation
          is nil, .organization called on nil (not handled)
        ✓ Already-used token → "This invite has already been used"
        ✓ Seat limit blocks invite when org is full

        1 bug found before merge.
```

## Who this is for

You already use Claude Code heavily and want consistent, high-rigor workflows instead of one mushy generic mode. You want to tell the model what kind of brain to use right now — founder taste, engineering rigor, paranoid review, or fast execution.

This is not a prompt pack for beginners. It is an operating system for people who ship.

## Install

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+. `/browse` compiles a native binary — works on macOS and Linux (x64 and arm64).

### Install — one command

```bash
git clone https://github.com/murtaza-bagwala/forge.git ~/.claude/skills/forge && cd ~/.claude/skills/forge && ./setup
```

That's it. The setup script handles everything automatically:
- Installs [Bun](https://bun.sh) if you don't have it
- Builds the `/browse` browser binary (~10 seconds)
- Downloads Chromium (~200MB, one-time)
- Registers all skills in Claude Code

Then open a **new Claude Code session** — the skills are live.

### Add to a project so teammates get it (optional)

```bash
cp -Rf ~/.claude/skills/forge .claude/skills/forge && rm -rf .claude/skills/forge/.git && cd .claude/skills/forge && ./setup
```

Commits the skill files to your repo. Teammates run `./setup` once after cloning — no other setup needed.

### What gets installed

- 6 skill files (plain Markdown) in `~/.claude/skills/`
- A headless browser binary for `/browse` and `/probe` (~58MB, gitignored)

Everything lives inside `.claude/`. Nothing touches your system PATH or runs in the background.

---

```
+----------------------------------------------------------------------------+
|                                                                            |
|   Are you a great software engineer who loves to write 10K LOC/day         |
|   and land 10 PRs a day like Murtaza?                                      |
|                                                                            |
|   Check out my work: github.com/murtaza-bagwala                            |
|                                                                            |
+----------------------------------------------------------------------------+
```

---

## How I use these skills

Created by [Murtaza Bagwala](https://github.com/murtaza-bagwala).

I built forge because I do not want AI coding tools stuck in one mushy mode.

Planning is not review. Review is not shipping. Founder taste is not engineering rigor. If you blur all of that together, you usually get a mediocre blend of all four.

I want explicit gears.

These skills let me tell the model what kind of brain I want right now. I can switch cognitive modes on demand — founder, eng manager, paranoid reviewer, release machine. That is the unlock.

---

## `/plan-product-review`

This is my **founder mode**.

This is where I want the model to think with taste, ambition, user empathy, and a long time horizon. I do not want it taking the request literally. I want it asking a more important question first:

**What is this product actually for?**

I think of this as **Brian Chesky mode**.

The point is not to implement the obvious ticket. The point is to rethink the problem from the user's point of view and find the version that feels inevitable, delightful, and maybe even a little magical.

### Example

I'm building a B2B SaaS tool and I say:

> "Add a way for users to cancel their subscription."

A weak assistant adds a cancel button.

That is not the real product.

In `/plan-product-review`, I want the model to ask whether "cancel" is even the right frame. The real job is reducing involuntary churn — most cancellations are triggered by confusion, lack of time, or failure to get team buy-in. Not actual dissatisfaction.

```
You:   Add a way for users to cancel their subscription.

You:   /plan-product-review

Claude: "Cancel subscription" is the last thing you want to build
        as a button. Here's the real product hiding inside it.

        Most B2B cancellations fall into three buckets:
        → "I don't have time to set it up" (40%)
        → "I can't get my team to adopt it" (35%)
        → "I found a cheaper alternative" (25%)

        A cancel button solves none of these. A smart offboarding
        flow can recover 20-30% of churners.

        The 10-star version:
        1. Trigger an exit survey on cancel click (2 questions max)
        2. Based on their answer, show a targeted rescue offer:
           - "No time" → offer a 1-on-1 onboarding call
           - "Team adoption" → offer a free team training session
           - "Too expensive" → offer a 2-month pause or downgrade
        3. If they still want to cancel: confirm, then keep access
           live until the billing period ends
        4. Send a reactivation email 3 days before access expires

        This is not more work than a cancel button.
        It's the same feature done in a way that actually serves
        both the user and the business.

        Questions before I build anything:
        → Do you have a free tier to downgrade to?
        → Does your billing provider (Stripe) support subscription
          pausing natively?
        → Who owns the cancel flow — the app or the billing portal?
```

That is what `/plan-product-review` does for me.

It does not just ask, "how do I add this feature?"
It asks, **"what is the 10-star product hiding inside this request?"**

---

## `/plan-eng-review`

This is my **eng manager mode**.

Once the product direction is right, I want a different kind of intelligence entirely. I do not want more sprawling ideation. I want the model to become my best technical lead.

This mode nails:

* architecture and system boundaries
* data flow and state transitions
* failure modes and edge cases
* trust boundaries
* test coverage and diagrams

One surprisingly big unlock: **it forces diagrams**.

LLMs get way more complete when you force them to draw the system. Diagrams make hand-wavy planning much harder to get away with.

### Example

The product direction is set: smart cancellation flow with exit survey, rescue offers, pause/downgrade, and end-of-period access. Now I need the technical spine.

```
You:   /plan-eng-review

Claude: SCOPE CHECK: Cancel flow touches billing (Stripe), subscriptions
        table, email jobs, and 3 new UI pages. That's 4 subsystems.
        Recommend SMALL CHANGE mode to keep the diff bisectable.

        Architecture:

          user clicks "Cancel"
                │
                ▼
          exit survey (2 questions)  →  answer stored in cancellation_requests
                │
                ▼
          rescue offer shown based on answer
                │
           ┌────┴────────────────────────────┐
           ▼                                 ▼
        "Pause 2 months"              "Still cancel"
        Stripe pause API              set cancel_at_period_end=true
        → subscription.status         → access until billing_period_end
          = 'paused'                  → reactivation email job scheduled

        State machine for subscriptions.status:
          active → cancellation_requested → cancelled
                ↘ paused ↗

        Issue 1: Where should cancellation_requests live?
        Recommend A) separate table (not a column on subscriptions).
        Why: survey answers + rescue offer shown + outcome are
        audit data. You will want this for cohort analysis.
        It doesn't belong mixed into the subscriptions billing row.

        Issue 2: When does Stripe get called?
        Recommend: immediately on "Still cancel" confirm, not on
        survey submit. Reason: user might abandon the rescue offer
        modal. Don't cancel until they explicitly confirm.

        Tests needed:
        - pause → reactivate flow (Stripe webhook round-trip)
        - cancel_at_period_end → access persists until billing_period_end
        - rescue email job enqueued with correct send_at timestamp
        - survey answer correctly routes to matching rescue offer
```

Not "make the idea smaller."
**Make the idea buildable.**

---

## `/audit`

This is my **paranoid staff engineer mode**.

Passing tests do not mean the branch is safe.

`/audit` exists because there is a whole class of bugs that survive CI and still punch you in production. This mode is not about dreaming bigger. It is about asking:

**What can still break?**

* N+1 queries
* race conditions
* bad trust boundaries
* missing auth checks
* escaping bugs
* broken invariants
* tests that pass while missing the real failure mode

### Example

The cancellation flow is implemented and tests are green. `/audit` runs anyway.

```
You:   /audit

Claude: 2 issues found. 1 critical.

        CRITICAL — cancel_at_period_end not re-verified on access check
        app/models/subscription.rb:89

        active? returns true if status = 'active' OR cancel_at_period_end
        = true. But there's no check that Time.now < billing_period_end.
        A subscription cancelled 3 months ago with an old billing_period_end
        still returns active? = true if the column was never cleared.

        Repro: cancel a subscription, manually set billing_period_end to a
        past date in the DB, call subscription.active? → returns true.

        Fix: add `billing_period_end > Time.current` to the active? guard,
        or set billing_period_end = nil on final cancellation webhook receipt.

        HIGH — Rescue offer can be shown multiple times
        app/controllers/subscriptions_controller.rb:34

        No guard on whether a rescue offer has already been shown and
        declined. User can hit browser back, resubmit the survey, and
        get a new rescue offer (e.g. another 2-month free pause on an
        account that already used its pause).

        Fix: add rescue_offer_shown_at and rescue_offer_declined_at
        columns to cancellation_requests. Skip the offer if
        rescue_offer_declined_at is non-null.
```

I do not want flattery here.
I want the model imagining the production incident before it happens.

---

## `/ship`

This is my **release machine mode**.

Once I have decided what to build, nailed the technical plan, and run a serious review, I do not want more talking. I want execution.

`/ship` is for the final mile. It is for a ready branch, not for deciding what to build.

A lot of branches die when the interesting work is done and only the boring release work is left. Humans procrastinate that part. AI should not.

### Example

The cancellation flow is done. Both audit issues are fixed.

```
You:   /ship

Claude: ── Pre-flight ───────────────────────────────────────────
        Branch: feat/cancellation-flow
        Merging origin/main → 1 commit behind, no conflicts.

        ── Tests ────────────────────────────────────────────────
        rspec spec/ — 247 examples, 0 failures (5.8s)

        ── Version ──────────────────────────────────────────────
        Bumping MICRO: 3.2.1 → 3.2.2

        ── CHANGELOG ────────────────────────────────────────────
        ## [3.2.2] - 2026-03-24
        ### Added
        - Smart cancellation flow with exit survey and rescue offers
        - Subscription pause (up to 2 months) via Stripe pause API
        - Reactivation email scheduled at billing_period_end - 3 days
        ### Fixed
        - active? guard now checks billing_period_end against current time
        - Rescue offer correctly shown at most once per cancellation event

        ── Shipping ─────────────────────────────────────────────
        [git commit -m "feat: smart cancellation flow with rescue offers (v3.2.2)"]
        [git push origin feat/cancellation-flow]
        [gh pr create --title "Smart cancellation flow (v3.2.2)"]

        PR #247 open → github.com/acme/app/pull/247
```

At this point I do not want more ideation.
I want the plane landed.

---

## `/probe`

This is my **QA lead mode**.

`/probe` gives the agent a testing methodology.

The most common use case: you're on a feature branch, you just finished coding, and you want to verify everything works. Just say `/probe` — it reads your git diff, identifies which pages and routes your changes affect, spins up the browser, and tests each one. No URL required. No manual test plan. It figures out what to test from the code you changed.

### Example

```
You:   /probe

Claude: Diff-aware mode: 9 files changed
        Affected routes: /settings/billing, /api/subscriptions/cancel,
                         /api/subscriptions/pause
        App on localhost:3000.

        Testing /settings/billing...
        ✓ "Cancel subscription" button visible for active subscribers
        ✓ Button hidden for users already in cancellation flow

        Testing exit survey...
        ✓ Survey renders after clicking Cancel
        ✓ All 3 answer options route to correct rescue offer
        ✓ "No thanks, still cancel" skips rescue and goes to confirm

        Testing rescue offers...
        ✓ "Pause for 2 months" — Stripe API called, status → paused,
          banner "Your subscription is paused until May 24" appears
        ✗ "Downgrade to free tier" — 422 error
          POST /api/subscriptions/downgrade →
          {"error": "Plan 'free' not found"}
          Repro: click Cancel → "Too expensive" → Downgrade → confirm
          Root cause: plan lookup uses `name` but DB stores `slug`
        ✓ Hard cancel → confirm modal → subscription marked cancelled
        ✓ Cancelled user retains access until billing_period_end

        1 bug found before merge. Pausing, downgrade, and hard cancel
        all need fixes or a green run before this ships.
```

Four modes:

- **Diff-aware** (automatic on feature branches) — reads `git diff main`, identifies affected pages, tests them. The fastest path from "I just wrote code" to "it works."
- **Full** — systematic exploration of the entire app. Documents 5-10 well-evidenced issues with screenshots.
- **Quick** (`--quick`) — 30-second smoke test. Homepage + top 5 nav targets. Loads? Console errors? Broken links?
- **Regression** (`--regression baseline.json`) — run full mode, diff against a previous baseline. Which issues are new? What's the score delta?

Reports and screenshots accumulate in `.forge/qa-reports/` so you can track quality over time.

---

## `/design-db`

This is my **database architect mode**.

Most projects run into database mistakes that are painful to fix after launch: wrong primary key type, no indexes on foreign keys, storing currency as a float, enums in comma-separated strings. `/design-db` forces the right decisions up front, interactively.

It works in five phases:

1. **Understand the problem** — asks about scale, consistency requirements, read/write ratio, and query patterns before touching schema
2. **SQL vs NoSQL decision** — runs a structured framework and recommends a database type with justification. If NoSQL, picks the right sub-type (document, graph, time-series, key-value, column-family)
3. **Schema design** — extracts entities, draws an ASCII ER diagram, then generates full DDL (SQL) or document shapes (NoSQL) with naming conventions, constraints, indexes, and embed/reference decisions
4. **Production checklist** — validates every table against data integrity, performance, operations, security, and scale criteria
5. **Watch-out list** — flags decisions that are fine now but will hurt at scale

### Example

```
You:   /design-db

Claude: Describe the system you're building.

You:   Real-time team chat. Workspaces, channels, threads,
       direct messages, reactions, read receipts. Aiming for
       50K MAU at launch, 10x growth in 18 months.

Claude: A few questions before I design anything:
        → Read/write ratio during peak hours?
        → Is full-text message search in scope?
        → How long is message history retained?
        → Per-user read receipts, or just last-seen per channel?

You:   Write-heavy during business hours. Search yes.
       History forever. Per-user read receipts.

Claude: Recommendation: PostgreSQL + Redis (hybrid)

        PostgreSQL for messages, channels, members, reactions.
        Reason: ACID guarantees — you cannot lose or double-deliver
        a message. Varied query patterns (history, search, threads)
        favor SQL flexibility over NoSQL's query-driven design.

        Redis for typing indicators, online presence, unread counts.
        Reason: these are high-write, ephemeral, and don't need
        durability. A Redis crash loses "user is typing" — acceptable.
        A Postgres crash losing messages — not acceptable.

        Entity diagram:

          Workspace (1) ────< Channel (N)
          Workspace (1) ────< Membership (N) >──── User (1)
          Channel (1) ────< Message (N)
          Message (1) ────< Message (N)   [threads via parent_id]
          Message (1) ────< Reaction (N)
          Channel (1) ────< ChannelRead (N) >──── User (1)

        [Generates full DDL — messages, channels, memberships,
        reactions, channel_reads with all indexes and constraints]

        Watch-outs:
        ⚠ messages will exceed 100M rows in 12 months — partition
          by workspace_id + DATE_TRUNC('month', created_at) from day one.
          Retrofitting partitioning on a live table is painful.
        ⚠ Full-text search on 100M rows: add tsvector column + GIN
          index now, or plan for Elasticsearch before you hit 10M rows.
        ⚠ channel_reads stores one row per user per channel.
          At 500K users × 50 channels = 25M rows. Use last_read_at
          (high-water mark) — never store one row per message read.
        ⚠ reactions.emoji stored as text CHECK constraint — add the
          allowed set now. Regexing emojis in production is miserable.

        Production checklist: 14/14 ✓
```

Anti-patterns it catches proactively: EAV tables, comma-separated arrays, currency as FLOAT, god tables (50+ columns), missing timestamps, indexes on nothing.

---

## Upgrading

Paste this into Claude Code:

> Update forge: run `cd ~/.claude/skills/forge && git fetch origin && git reset --hard origin/main && ./setup` (repo: https://github.com/murtaza-bagwala/forge). If this project also has forge at .claude/skills/forge, update it too: run `for s in browse plan-product-review plan-eng-review audit ship probe design-db; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/forge && cp -Rf ~/.claude/skills/forge .claude/skills/forge && rm -rf .claude/skills/forge/.git && cd .claude/skills/forge && ./setup`

The `setup` script rebuilds the browser binary and re-symlinks skills. It takes a few seconds.

## Uninstalling

Paste this into Claude Code:

> Uninstall forge: remove the skill symlinks by running `for s in browse plan-product-review plan-eng-review audit ship probe design-db; do rm -f ~/.claude/skills/$s; done` then run `rm -rf ~/.claude/skills/forge` and remove the forge section from CLAUDE.md. If this project also has forge at .claude/skills/forge, remove it by running `for s in browse plan-product-review plan-eng-review audit ship probe design-db; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/forge` and remove the forge section from the project CLAUDE.md too.

## Development

See [CONTRIBUTING.md](CONTRIBUTING.md) for setup, testing, and dev mode. See [ARCHITECTURE.md](ARCHITECTURE.md) for design decisions and system internals. See [BROWSER.md](BROWSER.md) for the browse command reference.

### Testing

```bash
bun test                     # free static tests (<5s)
EVALS=1 bun run test:evals   # full E2E + LLM evals (~$4, ~20min)
bun run eval:watch            # live dashboard during E2E runs
```

E2E tests stream real-time progress, write machine-readable diagnostics, and persist partial results that survive kills. See CONTRIBUTING.md for the full eval infrastructure.

## License

MIT
