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

### 4. Present candidates — then HARD STOP
Show each candidate concisely and readably:
- Order number (for easy selection) + name.
- One-sentence description.
- Category + a few relevant tags.
- Thumbnail: display it using the local path
  `${CLAUDE_PLUGIN_ROOT}/references/thumbs/<slug>.webp`. If the thumbnail file is missing
  or can't be displayed, DON'T fail — just present the text (name +
  description) and note "(thumbnail unavailable)".

If the match is weak (nothing really fits), **be honest**:
> "Nothing's a perfect fit for '<request>', these are the closest. Want me to
> search another category, or go ahead and adapt from one of these?"

Close with an explicit invitation to choose:
> "Which one do you want to use? Pick one or combine several (e.g. '#2 for the
> layout, take the colors from #4'). Or say 'none of these, search again'."

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
chosen slugs (in order) + the requested output format.

## What this skill must NOT do
- Write code / generate UI / create files — that's `build`'s job.
- Continue to `build` on its own without the user choosing (the hard stop is
  mandatory).
- Force 5 candidates when fewer are relevant.
- Claim "perfect fit" when the match is weak — be honest about match quality.
- Guess the category when the request has no signal at all — ask first.
- Handle "improve/fix my existing UI" requests — that's `review` + `glowup`.
