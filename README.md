# Forge

**Forge turns Claude Code from one generic assistant into a team of specialists you can summon on demand.**

Eight opinionated workflow skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Plan review, code review, one-command shipping, browser automation, QA testing, and engineering retrospectives — all as slash commands.

### Without forge

- The agent takes your request literally — it never asks if you're building the right thing
- It will implement exactly what you said, even when the real product is something bigger
- "Review my PR" gives inconsistent depth every time
- "Ship this" turns into a long back-and-forth about what to do
- The agent can write code but can't see your app — it's half blind
- You still do QA by hand: open browser, click around, check pages, squint at layouts

### With forge

| Skill | Mode | What it does |
|-------|------|--------------|
| `/plan-product-review` | Founder / CEO | Rethink the problem. Find the 10-star product hiding inside the request. |
| `/plan-eng-review` | Eng manager / tech lead | Lock in architecture, data flow, diagrams, edge cases, and tests. |
| `/audit` | Paranoid staff engineer | Find the bugs that pass CI but blow up in production. |
| `/ship` | Release engineer | Sync main, run tests, push, open PR. For a ready branch, not for deciding what to build. |
| `/browse` | QA engineer | Give the agent eyes. It logs in, clicks through your app, takes screenshots, catches breakage. Full QA pass in 60 seconds. |
| `/probe` | QA lead | Systematic QA testing. On a feature branch, auto-analyzes your diff, identifies affected pages, and tests them. Also: full exploration, quick smoke test, regression mode. |

## Demo: one feature, five modes

I start almost every feature in plan mode. I describe what I want to build, then use `/plan-product-review` to pressure-test whether I am even building the right thing. Only after the product direction is locked do I shift into engineering, review, ship, and test.

```
You:   [enter plan mode, describe the feature]
       I want to add seller photo upload to the listing app. Sellers
       should be able to upload a photo of their item and we create
       the listing from it.

You:   /plan-product-review

Claude: "Photo upload" is not the feature. The real job is helping sellers
        create listings that actually sell. Here's what the 10-star version
        looks like: auto-identify the product from the photo, pull specs and
        pricing comps from the web, draft the title and description, suggest
        the best hero image...

You:   /plan-eng-review

Claude: [Architecture diagram, state machine for upload → classify → enrich
        → draft pipeline, async job boundaries, failure modes, test matrix]

You:   [exit plan mode, implement the plan]

You:   /audit

Claude: Race condition: two tabs can overwrite cover-photo selection.
        Trust boundary: pulling web data into draft generation creates
        a prompt injection vector. Missing: orphan cleanup for failed uploads.

You:   (fix the issues)

You:   /ship

Claude: [Syncs main, runs tests, pushes branch, opens PR — 6 tool calls, done]

You:   /probe

Claude: Analyzing branch diff... 8 files changed, 3 routes affected.
        [Tests /listings/new, /listings/:id, /api/listings against localhost:3000]
        All 3 routes working. Upload + enrichment flow passes end to end.

You:   /probe https://staging.myapp.com --quick

Claude: [Smoke test: homepage + 5 pages, 30 seconds]
        Health Score: 91/100. No critical issues. 1 medium: mobile nav overlap.
```

## Who this is for

You already use Claude Code heavily and want consistent, high-rigor workflows instead of one mushy generic mode. You want to tell the model what kind of brain to use right now — founder taste, engineering rigor, paranoid review, or fast execution.

This is not a prompt pack for beginners. It is an operating system for people who ship.

## How to fly: 10 sessions at once

forge is powerful with one Claude Code session. It is transformative with ten.

[Conductor](https://conductor.build) runs multiple Claude Code sessions in parallel — each in its own isolated workspace. That means you can have one session running `/probe` on staging, another doing `/audit` on a PR, a third implementing a feature, and seven more working on other branches. All at the same time.

Each workspace gets its own isolated browser instance automatically — separate Chromium process, cookies, tabs, and logs stored in `.forge/` inside each project root. No port collisions, no shared state, no configuration needed. `/browse` and `/probe` sessions never interfere with each other, even across ten parallel workspaces.

This is the setup I use. One person, ten parallel agents, each with the right cognitive mode for its task. That is not incremental improvement. That is a different way of building software.

## Install

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+. `/browse` compiles a native binary — works on macOS and Linux (x64 and arm64).

### Step 1: Install on your machine

Open Claude Code and paste this. Claude will do the rest.

> Install forge: run `git clone https://github.com/murtaza-bagwala/forge.git ~/.claude/skills/forge && cd ~/.claude/skills/forge && ./setup` then add a "forge" section to CLAUDE.md that says to use the /browse skill from forge for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, and lists the available skills: /plan-product-review, /plan-eng-review, /audit, /ship, /browse, /probe. Then ask the user if they also want to add forge to the current project so teammates get it.

### Step 2: Add to your repo so teammates get it (optional)

> Add forge to this project: run `cp -Rf ~/.claude/skills/forge .claude/skills/forge && rm -rf .claude/skills/forge/.git && cd .claude/skills/forge && ./setup` then add a "forge" section to this project's CLAUDE.md that says to use the /browse skill from forge for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, lists the available skills: /plan-product-review, /plan-eng-review, /audit, /ship, /browse, /probe, and tells Claude that if forge skills aren't working, run `cd .claude/skills/forge && ./setup` to build the binary and register skills.

Real files get committed to your repo (not a submodule), so `git clone` just works. The binary and node\_modules are gitignored — teammates just need to run `cd .claude/skills/forge && ./setup` once to build (or `/browse` handles it automatically on first use).

### What gets installed

- Skill files (Markdown prompts) in `~/.claude/skills/forge/` (or `.claude/skills/forge/` for project installs)
- Symlinks at `~/.claude/skills/browse`, `~/.claude/skills/probe`, `~/.claude/skills/audit`, etc. pointing into the forge directory
- Browser binary at `browse/dist/browse` (~58MB, gitignored)
- `node_modules/` (gitignored)

Everything lives inside `.claude/`. Nothing touches your PATH or runs in the background.

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

## `/browse`

This is my **QA engineer mode**.

`/browse` is the skill that closes the loop. Before it, the agent could think and code but was still half blind. It had to guess about UI state, auth flows, redirects, console errors, empty states, and broken layouts. Now it can just go look.

That is a real step-change. The full cycle becomes: plan, code, run the app, inspect the UI, reproduce the bug, verify the fix, ship. QA stops being "go click around manually" and becomes part of the same agentic loop. It gives the agent eyes. Now it can do the boring, high-context QA work itself: click through the app, catch breakage, verify the fix, and keep going. That feels like having a real operator in the machine.

It is a compiled binary that talks to a persistent Chromium daemon — built on [Playwright](https://playwright.dev/) by Microsoft. First call starts the browser (~3s). Every call after that: ~100-200ms. The browser stays running between commands, so cookies, tabs, and localStorage carry over.

After I push a branch, I tell Claude to go check staging. It does the entire QA pass for me:

```
You:   /browse staging.myapp.com — log in, test the signup flow, and check
       every page I changed in this branch

Claude: [18 tool calls, ~60 seconds]

        > browse goto https://staging.myapp.com/signup
        > browse snapshot -i
        > browse fill @e2 "test@example.com"
        > browse fill @e3 "password123"
        > browse click @e5                    (Submit)
        > browse screenshot /tmp/signup.png
        > Read /tmp/signup.png

        Signup works. Redirected to onboarding. Now checking changed pages.

        > browse goto https://staging.myapp.com/dashboard
        > browse screenshot /tmp/dashboard.png
        > Read /tmp/dashboard.png
        > browse console

        Dashboard loads. No console errors. Charts render with sample data.

        > browse goto https://staging.myapp.com/settings/billing
        > browse screenshot /tmp/billing.png
        > Read /tmp/billing.png

        Billing page loads. Plan selector shows Free/Pro/Enterprise.
        Stripe elements render correctly.

        > browse goto https://staging.myapp.com/api/health
        > browse text

        API health endpoint returns {"status":"ok","version":"2.4.1"}.

        All 4 pages load correctly. No console errors. No broken layouts.
        Signup → onboarding → dashboard flow works end to end.
```

18 tool calls, about a minute. It signed up a test user, navigated every page I changed, took screenshots, read them, checked the console for errors, and verified the API. That is a full QA pass. I did not open a browser.

It works for reading documentation, verifying deployments, filling out forms, taking screenshots, comparing pages across environments, and anything else where Claude needs eyes on a live URL.

**Security note:** `/browse` runs a persistent Chromium session. Cookies, localStorage, and session state carry over between commands. Do not use it against sensitive production environments unless you intend to — it is a real browser with real state. The session auto-shuts down after 30 minutes of idle time.

For the full command reference, technical internals, and architecture details, see [BROWSER.md](BROWSER.md).

---

## `/probe`

This is my **QA lead mode**.

`/browse` gives the agent eyes. `/probe` gives it a testing methodology.

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

## Troubleshooting

**Skill not showing up in Claude Code?**
Run `cd ~/.claude/skills/forge && ./setup` (or `cd .claude/skills/forge && ./setup` for project installs). This rebuilds symlinks so Claude can discover the skills.

**`/browse` fails or binary not found?**
Run `cd ~/.claude/skills/forge && bun install && bun run build`. This compiles the browser binary. Requires Bun v1.0+.

**Project copy is stale?**
Re-copy from global: `for s in browse plan-product-review plan-eng-review audit ship test; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/forge && cp -Rf ~/.claude/skills/forge .claude/skills/forge && rm -rf .claude/skills/forge/.git && cd .claude/skills/forge && ./setup`

**`bun` not installed?**
Install it: `curl -fsSL https://bun.sh/install | bash`

## Upgrading

Paste this into Claude Code:

> Update forge: run `cd ~/.claude/skills/forge && git fetch origin && git reset --hard origin/main && ./setup` (repo: https://github.com/murtaza-bagwala/forge). If this project also has forge at .claude/skills/forge, update it too: run `for s in browse plan-product-review plan-eng-review audit ship test; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/forge && cp -Rf ~/.claude/skills/forge .claude/skills/forge && rm -rf .claude/skills/forge/.git && cd .claude/skills/forge && ./setup`

The `setup` script rebuilds the browser binary and re-symlinks skills. It takes a few seconds.

## Uninstalling

Paste this into Claude Code:

> Uninstall forge: remove the skill symlinks by running `for s in browse plan-product-review plan-eng-review audit ship test; do rm -f ~/.claude/skills/$s; done` then run `rm -rf ~/.claude/skills/forge` and remove the forge section from CLAUDE.md. If this project also has forge at .claude/skills/forge, remove it by running `for s in browse plan-product-review plan-eng-review audit ship test; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/forge` and remove the forge section from the project CLAUDE.md too.

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
