---
name: brainstorming
description: Brainstorm a UI by picking design references from the getokui taste library (213 curated landing pages). Invoked whenever the user asks to build/develop/design a NEW UI via getokui — a landing page in any vertical (AI/SaaS, fintech, crypto/web3, dev-tools, real-estate, architecture, portfolio, restaurant, wellness, luxury, travel, music, cinematic, gaming, agency), e.g. "getokui, build a SaaS landing page", "getokui brainstorm a fintech landing", "getokui design a portfolio in react". This skill reads the local index.json, filters & ranks candidates, then presents the 5 BEST CANDIDATES (name + description + thumbnail) and STOPS to wait for the user to choose (may pick more than one, may reject all). This skill ONLY brainstorms & picks — it does NOT write code or generate UI (that's the build skill). Do not start building UI for any getokui "make something new" request before going through this skill. (For improving an existing UI file, use the review + glowup skills instead.)
---

# getokui brainstorming — Suggest References & Checkpoint

This skill's job: turn the user's request into the **5 most relevant reference
candidates** from the local taste library, present them, then **stop** and let
the user decide. This is getokui's agentic checkpoint — there's reasoning
(ranking + a little ideation on direction), but the user stays in control.

This is the **create-from-scratch** entry point. (To improve a UI file that
already exists, use `review` → `glowup` instead.)

This skill does **not** write code. Only after the user picks do you continue to
the `build` skill.

## READ FIRST — shared doctrine (mandatory)

Read `${CLAUDE_PLUGIN_ROOT}/skills/shared/DOCTRINE.md` and follow it. For
brainstorming the load-bearing parts are:
- **§0 reasoning pre-flight** — restate the goal + mood before you rank, so the
  5 candidates are actually targeted, not a generic category dump.
- **§2b web layer** — alongside the local candidates, do a quick current-year
  design scan + name famous-brand benchmarks for the vertical (see Step 3.5).
- **§4 communication** — present candidates as a warm human summary with clear
  numbered options and a real question; NO emoji anywhere.
- **§5 checkpoint** — the HARD STOP is sacred; options + question, then wait.

You don't build here, so §1/§3 (icons/shadcn) mostly land in `build` — but if the
user mentions an icon set, shadcn, a specific animation, or a brand they want the
feel of, note it and carry it into the handoff so `build` honors it.

## Prerequisite

The library ships **inside this plugin** at `${CLAUDE_PLUGIN_ROOT}/references/`
— it's bundled, so it's present as soon as the plugin is installed (no separate
clone). Sanity-check that `${CLAUDE_PLUGIN_ROOT}/references/index.json` exists:
- **Present** → continue to Step 1.
- **Missing** (unexpected — broken install) → tell the user the bundled library
  wasn't found and suggest reinstalling the plugin (`/plugin install`). Don't
  proceed on a guess.

## Steps

### 1. Parse the user's request
From the user's sentence, determine:
- **Primary category** — match against the `category` field in the index. The
  library is **213 landing pages across verticals**. Categories present:
  `ai-saas`, `saas`, `fintech`, `crypto-web3`, `dev-tools`, `real-estate`,
  `architecture`, `portfolio`, `restaurant`, `wellness`, `luxury`, `travel`,
  `music`, `cinematic`, `gaming`, `agency`, and `tech` (generic/fallback).
- **Tags/mood** mentioned (e.g. "dark", "minimal", "saas", "fintech",
  "gradient") — match against `tags`, `description`, `colors`.
- **Output format** if mentioned (`html` default, `react`, `next`). Save this
  to pass on to `build` — BUT don't generate anything now.

If the request is too vague (unclear what kind of landing they want), ask one
short question first: "what kind of landing do you want? (e.g. SaaS / fintech /
restaurant / portfolio)". Don't guess the category if there's genuinely no
signal.

### 2. Read & filter index.json
Read `${CLAUDE_PLUGIN_ROOT}/references/index.json`. Filter `templates[]`:
1. Priority 1 — `category` matches exactly what was requested.
2. Priority 2 — if same-category candidates < 5, fill in with ones whose
   `tags`/`description` overlap the mood the user mentioned.

### 3. Rank → take the top 5
Order candidates by relevance. Ranking signals (strongest to weakest):
- Exact `category` match.
- Number of `tags` overlapping the request.
- Mood match in `description` (e.g. user asks "dark & minimal" → favor ones
  described as dark/minimal).
- `colors` match if the user mentioned a color.

Take **at most 5**. If only 3 are genuinely relevant, present just 3 — don't
force in irrelevant ones to reach 5.

### 3.5. Scan the web for current-year direction + brand benchmarks (doctrine §2b)
The local library is frozen at bundle time — the web keeps the direction current.
Before presenting, do a quick pass:
- Read `${CLAUDE_PLUGIN_ROOT}/skills/shared/WEB-BENCHMARKS.md` for the famous
  brands mapped to this vertical (Indonesian + global).
- If web access is available, run 1–2 quick searches with the **current year** to
  see what great looks like now — e.g. "fintech landing page design 2026",
  `site:awwwards.com <vertical>`, `site:dribbble.com <vertical> 2026`, or a
  specific brand's site (Awwwards, Dribbble, Behance, Godly all work). Judge with
  getokui's eyes (composition, motion, component polish).
- Pick **1–2 named benchmarks** to mention as the taste bar (e.g. "for a payment
  vertical, we'd aim for the CTA/trust feel of **Xendit** and **Stripe**"; for
  hajj/umrah, the trust + warmth of **Traveloka** + an umrah operator; for a
  school, **Ruangguru**; for a gov/public service, the clarity of **GOV.UK**).
- **Keep the source URLs.** As you scan, collect up to **5 "sources of truth"** —
  the real gallery/brand pages you actually looked at (Awwwards entry, Dribbble
  shot, a live brand site, etc.). Save each as a clickable `https://` link with a
  one-line note on what's worth stealing from it. You'll present these alongside
  the 5 local references in Step 4 so the user can open both.

This is direction only — you're not extracting assets, and you still don't build
here. If offline or search fails, use `WEB-BENCHMARKS.md`'s named brands and move
on — never block on the web (present fewer than 5 sources, or none, rather than
inventing URLs — never fabricate a link).

### 4. Present candidates — then HARD STOP
Show each candidate concisely and readably:
- Order number (for easy selection) + name.
- One-sentence description.
- Category + a few relevant tags.
- **Preview link** (the full rendered page, not just the thumb): emit a
  clickable `file://` URL to the actual reference HTML so the user can open it in
  a browser. Build it from the resolved plugin root:
  1. Resolve `${CLAUDE_PLUGIN_ROOT}` to its real absolute path (the same value you
     already use for the thumbnail).
  2. Convert backslashes to forward slashes (Windows) and prefix with `file:///`.
  3. Point it at `references/templates/<slug>.html`.

  So it renders as one line per candidate, e.g.:
  `Preview: file:///C:/Users/user/.claude/plugins/.../references/templates/aero-studio.html`

  Put the `Preview:` line directly above the thumbnail — the link is the new hero
  of this step (full page), the thumbnail is the quick glance under it. If the
  HTML file is missing or the path can't be resolved, DON'T fail — skip the
  preview line for that candidate and note "(preview unavailable)".
- Thumbnail: display it using the local path
  `${CLAUDE_PLUGIN_ROOT}/references/thumbs/<slug>.webp`. If the thumbnail file is missing
  or can't be displayed, DON'T fail — just present the text (name +
  description) and note "(thumbnail unavailable)".

Open with a one-line human summary of the direction you read from their brief
(doctrine §4), e.g. "Buat AI SaaS yang dark + premium, ini 5 yang paling nyetel:"
— then the numbered list. No emoji.

After the list, add a one-line **web benchmark** note from Step 3.5 so the user
sees the current-year taste bar, e.g. "Buat kualitas komponen & CTA-nya, kita
patok ke **Xendit** + **Stripe** (2026) — pola-nya diadaptasi ke brand lo, bukan
di-copy." Keep it short; it's context, not another menu.

Then, under a short header like **"Sources of truth (web):"**, list the up-to-5
web sources you collected in Step 3.5 — each as a clickable `https://` link + a
few words on what to steal from it. This mirrors the 5 references: local refs =
previewable HTML you'll extract from, web sources = the current-year taste bar.
Example:
> Sources of truth (web):
> 1. https://awwwards.com/... — hero motion + scroll reveal
> 2. https://dribbble.com/... — bento pricing layout
>
> Skip this block entirely if you're offline / found nothing — don't pad it with
> invented links.

If the match is weak (nothing really fits), **be honest**:
> "Nothing's a perfect fit for '<request>', these are the closest. Want me to
> search another category, or go ahead and adapt from one of these?"

Close with an explicit invitation to choose:
> "Which one do you want to use? Pick one or combine several (e.g. '#2 for the
> layout, take the colors from #4'). Or say 'none of these, search again'."

If the user named an icon set (Lucide/Solar), shadcn, or a specific animation,
acknowledge it here and remember it for the `build` handoff.

**HARD STOP here.** Do NOT continue to `build`, do NOT start writing UI on your
own assumption. Wait for the user's explicit answer in the next message — even
if it seems "obvious" which one they'll pick.

### 5. Interpret the user's answer (next turn)
When the user replies:
- **Pick 1** → record its slug. Continue to `build` (confirm format if not yet
  clear).
- **Pick several** → record the order. First pick = base layout, the rest =
  style sources. Continue to `build`.
- **Reject all / "search again"** → go back to Step 2 with looser filters or a
  different category. Don't force the old candidates.
- **Ambiguous** (e.g. "the blue one" but there are two) → ask a short
  confirmation, don't guess.

Once the choice is clear, hand off to the `build` skill: bring the list of
chosen slugs (in order) + the requested output format + any icon-set / shadcn /
animation preferences the user mentioned + the **web benchmark(s)** you named in
Step 3.5 so `build` matches that component/CTA bar.

## What this skill must NOT do
- Write code / generate UI / create files — that's `build`'s job.
- Continue to `build` on its own without the user choosing (the hard stop is
  mandatory).
- Force 5 candidates when fewer are relevant.
- Claim "perfect fit" when the match is weak — be honest about match quality.
- Guess the category when the request has no signal at all — ask first.
- Handle "improve/fix my existing UI" requests — that's `review` + `glowup`.
