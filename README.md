# Forge

**Forge turns Claude Code from one generic assistant into a team of specialists you can summon on demand.**

Eight opinionated workflow skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Plan review, code review, database design, one-command shipping, browser automation, and QA testing — all as slash commands.

### Without forge

- The agent takes your request literally — it never asks if you're building the right thing
- It will implement exactly what you said, even when the real product is something bigger
- "Review my PR" gives inconsistent depth every time
- "Ship this" turns into a long back-and-forth about what to do
- You still do QA by hand: open browser, click around, check pages, squint at layouts

### With forge

| Skill | Mode | What it does |
|-------|------|--------------|
| `/forge` | Help & index | List all available skills with descriptions and quick-start examples. Start here. |
| `/plan-product-review` | Founder / CEO | Rethink the problem. Find the 10-star product hiding inside the request. |
| `/plan-eng-review` | Eng manager / tech lead | Lock in architecture, data flow, diagrams, edge cases, and tests. |
| `/audit` | Paranoid staff engineer | Find the bugs that pass CI but blow up in production. |
| `/ship` | Release engineer | Sync main, run tests, push, open PR. For a ready branch, not for deciding what to build. |
| `/probe` | QA lead | Systematic QA testing. On a feature branch, auto-analyzes your diff, identifies affected pages, and tests them. Also: full exploration, quick smoke test, regression mode. |
| `/design-db` | Database architect | Interactive database design workshop. SQL vs NoSQL decision framework, ER diagrams, full schema with indexes and constraints, production checklist, anti-pattern callouts. |

## Demo: one feature, five modes

The app: **quick-notes** — a simple Bun note-taking app with a form to add notes,
a list view, a delete button per note, and a `/api/notes` JSON endpoint.

The feature request: add a search bar so users can filter notes by title or content.

```
You:   I want to add a search bar to quick-notes so users can filter
       notes by title or content as they type.

You:   /plan-product-review

Claude: "Search bar" undersells the feature. The real job is helping
        users find the right note fast — especially once notes pile up.
        The 10-star version feels instant: type a word, notes filter in
        real time with no page reload, matched keyword highlighted, and
        a count showing "3 of 12 notes". Empty query restores all notes.
        That's not more work — it's the same feature done right.

You:   /plan-eng-review

Claude: Client-side JS filter — no new server routes needed.
        Implementation: input event listener → filter fn → toggle
        .hidden class on note cards. Case-insensitive match across
        title + body. Highlight matches with <mark> tags. Edge cases:
        empty query → show all, no matches → show "nothing found" state.
        15-20 lines of vanilla JS added inline to server.ts.

You:   [implement the plan — add search input + filter script to server.ts]

You:   /audit

Claude: XSS: n.title and n.body are injected directly into innerHTML
        with no escaping — a note body containing <script>alert(1)</script>
        executes in the browser. Fix: escape HTML before rendering.
        Also missing: empty state message when search finds no matches.

You:   (fix XSS, add empty search state)

You:   /ship

Claude: [Runs bun test, commits "add real-time note search with XSS fix",
        pushes branch, opens PR — done in 5 tool calls]

You:   /probe

Claude: Analyzing diff... 1 file changed: server.ts
        App detected on localhost:3000.

        ✓ Homepage loads, add-note form renders
        ✓ Created 3 notes — all appear correctly
        ✓ Search "meeting" filters to matching notes instantly
        ✓ Clearing search restores all notes
        ✓ Search with no matches shows "nothing found" state
        ✓ XSS fix verified — <script> tags render as escaped text
        ✓ No console errors

        All flows passing. Ready to merge.
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

Say I am building a Craigslist-style listing app and I say:

> "Let sellers upload a photo for their item."

A weak assistant will add a file picker and save an image.

That is not the real product.

In `/plan-product-review`, I want the model to ask whether "photo upload" is even the feature. Maybe the real feature is helping someone create a listing that actually sells.

If that is the real job, the whole plan changes.

Now the model should ask:

* Can we identify the product from the photo?
* Can we infer the SKU or model number?
* Can we search the web and draft the title and description automatically?
* Can we pull specs, category, and pricing comps?
* Can we suggest which photo will convert best as the hero image?
* Can we detect when the uploaded photo is ugly, dark, cluttered, or low-trust?
* Can we make the experience feel premium instead of like a dead form from 2007?

That is what `/plan-product-review` does for me.

It does not just ask, "how do I add this feature?"
It asks, **"what is the 10-star product hiding inside this request?"**

That is a very different kind of power.

---

## `/plan-eng-review`

This is my **eng manager mode**.

Once the product direction is right, I want a different kind of intelligence entirely. I do not want more sprawling ideation. I do not want more "wouldn't it be cool if." I want the model to become my best technical lead.

This mode should nail:

* architecture
* system boundaries
* data flow
* state transitions
* failure modes
* edge cases
* trust boundaries
* test coverage

And one surprisingly big unlock for me: **diagrams**.

LLMs get way more complete when you force them to draw the system. Sequence diagrams, state diagrams, component diagrams, data-flow diagrams, even test matrices. Diagrams force hidden assumptions into the open. They make hand-wavy planning much harder.

So `/plan-eng-review` is where I want the model to build the technical spine that can carry the product vision.

### Example

Take the same listing app example.

Let's say `/plan-product-review` already did its job. We decided the real feature is not just photo upload. It is a smart listing flow that:

* uploads photos
* identifies the product
* enriches the listing from the web
* drafts a strong title and description
* suggests the best hero image

Now `/plan-eng-review` takes over.

Now I want the model to answer questions like:

* What is the architecture for upload, classification, enrichment, and draft generation?
* Which steps happen synchronously, and which go to background jobs?
* Where are the boundaries between app server, object storage, vision model, search/enrichment APIs, and the listing database?
* What happens if upload succeeds but enrichment fails?
* What happens if product identification is low-confidence?
* How do retries work?
* How do we prevent duplicate jobs?
* What gets persisted when, and what can be safely recomputed?

And this is where I want diagrams — architecture diagrams, state models, data-flow diagrams, test matrices. Diagrams force hidden assumptions into the open. They make hand-wavy planning much harder.

That is `/plan-eng-review`.

Not "make the idea smaller."
**Make the idea buildable.**

---

## `/audit`

This is my **paranoid staff engineer mode**.

Passing tests do not mean the branch is safe.

`/audit` exists because there is a whole class of bugs that can survive CI and still punch you in the face in production. This mode is not about dreaming bigger. It is not about making the plan prettier. It is about asking:

**What can still break?**

This is a structural audit, not a style nitpick pass. I want the model to look for things like:

* N+1 queries
* stale reads
* race conditions
* bad trust boundaries
* missing indexes
* escaping bugs
* broken invariants
* bad retry logic
* tests that pass while missing the real failure mode

### Example

Suppose the smart listing flow is implemented and the tests are green.

`/audit` should still ask:

* Did I introduce an N+1 query when rendering listing photos or draft suggestions?
* Am I trusting client-provided file metadata instead of validating the actual file?
* Can two tabs race and overwrite cover-photo selection or item details?
* Do failed uploads leave orphaned files in storage forever?
* Can the "exactly one hero image" rule break under concurrency?
* If enrichment APIs partially fail, do I degrade gracefully or save garbage?
* Did I accidentally create a prompt injection or trust-boundary problem by pulling web data into draft generation?

That is the point of `/audit`.

I do not want flattery here.
I want the model imagining the production incident before it happens.

---

## `/ship`

This is my **release machine mode**.

Once I have decided what to build, nailed the technical plan, and run a serious review, I do not want more talking. I want execution.

`/ship` is for the final mile. It is for a ready branch, not for deciding what to build.

This is where the model should stop behaving like a brainstorm partner and start behaving like a disciplined release engineer: sync with main, run the right tests, make sure the branch state is sane, update changelog or versioning if the repo expects it, push, and create or update the PR.

Momentum matters here.

A lot of branches die when the interesting work is done and only the boring release work is left. Humans procrastinate that part. AI should not.

### Example

Suppose the smart listing flow is finished.

The product thinking is done.
The architecture is done.
The review pass is done.
Now the branch just needs to get landed.

That is what `/ship` is for.

It takes care of the repetitive release hygiene so I do not bleed energy on:

* syncing with main
* rerunning tests
* checking for weird branch state
* updating changelog/version metadata
* pushing the branch
* opening or updating the PR

At this point I do not want more ideation.
I want the plane landed.

---

## `/probe`

This is my **QA lead mode**.

`/probe` gives the agent a testing methodology.

The most common use case: you're on a feature branch, you just finished coding, and you want to verify everything works. Just say `/probe` — it reads your git diff, identifies which pages and routes your changes affect, spins up the browser, and tests each one. No URL required. No manual test plan. It figures out what to test from the code you changed.

```
You:   /probe

Claude: Analyzing branch diff against main...
        12 files changed: 3 controllers, 2 views, 4 services, 3 tests

        Affected routes: /listings/new, /listings/:id, /api/listings
        Detected app running on localhost:3000.

        [Tests each affected page — navigates, fills forms, clicks buttons,
        screenshots, checks console errors]

        QA Report: 3 routes tested, all working.
        - /listings/new: upload + enrichment flow works end to end
        - /listings/:id: detail page renders correctly
        - /api/listings: returns 200 with expected shape
        No console errors. No regressions on adjacent pages.
```

Four modes:

- **Diff-aware** (automatic on feature branches) — reads `git diff main`, identifies affected pages, tests them specifically. The fastest path from "I just wrote code" to "it works."
- **Full** — systematic exploration of the entire app. 5-15 minutes depending on app size. Documents 5-10 well-evidenced issues.
- **Quick** (`--quick`) — 30-second smoke test. Homepage + top 5 nav targets. Loads? Console errors? Broken links?
- **Regression** (`--regression baseline.json`) — run full mode, then diff against a previous baseline. Which issues are fixed? Which are new? What's the score delta?

```
You:   /probe https://staging.myapp.com

Claude: [Explores 12 pages, fills 3 forms, tests 2 flows]

        QA Report: staging.myapp.com — Health Score: 72/100

        Top 3 Issues:
        1. CRITICAL: Checkout form submits with empty required fields
        2. HIGH: Mobile nav menu doesn't close after selecting an item
        3. MEDIUM: Dashboard chart overlaps sidebar below 1024px

        [Full report with screenshots saved to .forge/qa-reports/]
```

Reports and screenshots accumulate in `.forge/qa-reports/` so you can track quality over time and compare runs.

---

## `/design-db`

This is my **database architect mode**.

Most projects run into database mistakes that are painful to fix after launch: wrong primary key type, no indexes on foreign keys, storing currency as a float, enums in comma-separated strings. `/design-db` forces the right decisions up front, interactively.

It works in five phases:

1. **Understand the problem** — asks about scale, consistency requirements, read/write ratio, and query patterns before touching schema
2. **SQL vs NoSQL decision** — runs a structured framework and recommends a database type with justification. If NoSQL, picks the right sub-type (document, graph, time-series, key-value, column-family)
3. **Schema design** — extracts entities, draws an ASCII ER diagram, then generates full DDL (SQL) or document shapes (NoSQL) with naming conventions, constraints, indexes, and embed/reference decisions
4. **Production checklist** — validates every table against data integrity, performance, operations, security, and scale criteria
5. **Watch-out list** — flags decisions that are fine now but will hurt at scale (e.g. INT primary key hitting the 2B ceiling)

```
You:   /design-db

Claude: Describe the system you're building. What does it do,
        who uses it, and what are the core operations?

You:   A marketplace where freelancers list services and clients
       book them. Payments go through Stripe. ~10K users at launch.

Claude: A few questions before I design anything...
        [Asks about scale, consistency needs, query patterns]

        Recommendation: PostgreSQL
        Payments + bookings need ACID transactions. Query patterns
        are varied (search, filters, user history). Schema is known
        upfront. Strong team SQL fluency beats a NoSQL learning curve
        at your scale.

        Entity map:
          User (1) ────< Service (N)
          User (1) ────< Booking (N) >──── Service (1)
          Booking (1) ──── Payment (1)

        [Generates full DDL with indexes, constraints, soft deletes]

        Production checklist: 12/12 ✓
        Watch-out: services.price stored as NUMERIC(10,2) ✓
                   booking_id uses UUID (distributed-safe) ✓
```

Anti-patterns it catches proactively: EAV tables, comma-separated arrays, currency as FLOAT, god tables, missing timestamps, no indexes anywhere.

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
