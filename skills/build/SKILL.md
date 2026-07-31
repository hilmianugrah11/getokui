---
name: build
description: Adapt the getokui reference(s) the user picked into the user's own UI, in the requested format (HTML default, React 19, or Next.js 15). Invoked after the brainstorming skill once the user has chosen one or more candidates. This skill reads the chosen template file(s) from the local library, TAKES the visual style & structure (NOT the template's original text content), mixes when there are several picks (first pick = base layout, the rest = style sources), converts to the target format, then writes the output file. Do not invoke this skill before the user has chosen via brainstorming.
---

# getokui build — Adapt into the User's Own UI

This skill's job: turn the chosen reference(s) into real UI code that belongs to
the user — not a raw copy. Take the **style, structure & motion**, fill with the
**user's content** (or clear placeholders), then emit in the requested format.

## READ FIRST — shared doctrine (mandatory)

Before anything else, read `${CLAUDE_PLUGIN_ROOT}/skills/shared/DOCTRINE.md` and
follow it. It governs the things this skill depends on:
- **§0 reasoning pre-flight** — do it before you generate a single line.
- **§1 icons** — NO emoji, ever; use Lucide (tech/clean) or Solar (premium/soft).
- **§2 reference extraction** — read the pre-extracted **Design DNA file**
  (`references/dna/<slug>.json`) FIRST, then the HTML. The DNA hands you the real
  class strings, spacing, and `@keyframes` so you copy instead of invent.
  Inventing from scratch is the failure mode this whole plugin exists to prevent.
- **§2b web layer** — also match a current-year, famous-brand quality bar for the
  component/CTA polish (Xendit, Stripe, Linear, Bank Jago, Gojek…); adapt the
  pattern, never copy the brand's logo/copy/assets. See Step 1.6 +
  `skills/shared/WEB-BENCHMARKS.md`.
- **§3 shadcn/ui** — use real shadcn for React/Next interactive components,
  its patterns for HTML, always re-skinned to the reference palette.
- **§4 communication** — human summary + options + a next-step question.
- **§6 hard floors + §6a anti-slop** — reproduce the reference's real
  `layout.hero_layout` (NOT the generic centered-hero), land ≥1 signature move,
  add asymmetry; plus `py-20`+ sections, `text-5xl`+ hero, ≥2 real motions,
  0 emoji, one radius + one shadow token. Non-negotiable.
- **§7 proof-of-extraction gate** — show the "Design DNA I'm using" block BEFORE
  you write any UI.
- **§8 self-check** — run it before you write files.

The steps below are the *what*; the doctrine is the *how*. Both are required.

Prerequisite: the user already chose candidates via the `brainstorming` skill.
If not, go back to `brainstorming` first.

## Input carried over from `brainstorming`
- List of chosen slugs, **in order** (first pick = base, the rest = style
  sources).
- Output format: `html` (default) / `react` / `next`.
- Any content context from the user (e.g. product name, button text).
- Any **web benchmark(s)** brainstorming named (brands whose component/CTA feel
  to match) + any icon-set / shadcn / animation preferences.

## Steps

### 0. Reasoning pre-flight (doctrine §0) — do NOT skip
Before reading files or writing anything, run the pre-flight from the
doctrine and show the user a short version:
1. Restate the goal in one sentence.
2. Name the exact reference slug(s) + the specific thing you'll pull from each.
3. List the sections/components you'll build, top to bottom.
4. Pick the icon set (Lucide or Solar) + one-line reason.
5. Note any component that needs a shadcn/ui pattern.
6. Name the biggest AI-slop risk and how you'll avoid it.
7. Inspect the project's real structure and name the exact path each file will
   land in — match the project's folders/naming/styling, don't invent (§9 +
   Step 3.5 below).

This is what makes Sonnet's output look designed instead of templated. If the
brief is missing something you need (format, mood, content), ask now (doctrine
§4) — don't guess.

### 1. Read the Design DNA FIRST, then the template — and EXTRACT, don't skim
For each chosen slug, in this order:

1. **Read the DNA file** `${CLAUDE_PLUGIN_ROOT}/references/dna/<slug>.json`. This
   is pre-extracted for you and is the fastest, most reliable source of the real
   tokens (doctrine §2 "Start from the pre-extracted DNA"). Pull from it:
   - `hero.h1_classes` → your headline's exact size/weight/tracking/leading.
   - `hero.cta_classes` → your primary button's exact padding/radius/shadow/hover.
   - `spacing.section_padding` / `container_max` / `gap` → your vertical rhythm.
   - `radius` + `shadow` → pick one of each and reuse everywhere (doctrine §6).
   - `layout.hero_layout` → build THIS composition (split / asymmetric /
     centered), not your default centered hero (doctrine §6a — anti-slop).
   - `layout.composition_techniques` → pull ≥1 through as your signature move
     (marquee, bento, rotated/overlap, horizontal-scroll, grain, etc.).
   - `motion.keyframes_css` → paste these `@keyframes` verbatim into your output.
   - `motion.animate_classes` / `techniques` / `signature` → the feel to preserve.
2. **Read the HTML** `${CLAUDE_PLUGIN_ROOT}/references/templates/<slug>.html` for
   anything the DNA doesn't capture — a component's fuller inner structure, the
   exact scroll-observer script, the gradient stops. The DNA is the map; the HTML
   is the territory.
3. **Read the index entry** in `${CLAUDE_PLUGIN_ROOT}/references/index.json` for
   `colors`, `fonts`, `sections`.

You are not reading for vibes — you are **extracting concrete material**. The DNA
already did the hard part; there is no valid reason to substitute an invented
value for a populated DNA field.

If the DNA or HTML file is missing → tell the user, offer to go back to
`brainstorming` to choose another. Don't proceed on a guess.

### 1.6. Web benchmark pass (doctrine §2b) — the current-year quality bar
The local DNA is your concrete, legal-to-copy anchor (real class strings +
`@keyframes`). The web layer sits ON TOP of it and raises the bar on component &
CTA polish to a famous-brand, current-year standard. Do this before the proof
gate so the benchmark shows up in the "Design DNA I'm using" block:

1. **Read** `${CLAUDE_PLUGIN_ROOT}/skills/shared/WEB-BENCHMARKS.md` and find the
   1–2 named brands mapped to this vertical (Indonesian + global) — e.g. payment
   → Xendit / Midtrans / DOKU + Stripe; super-app/marketplace → Gojek / Grab /
   Tokopedia + Airbnb; telco/hosting → Telkomsel / Niagahoster + Cloudflare;
   ai-saas/dev-tools → Linear / Vercel + Ruangguru; fintech → Bank Jago / Jenius
   + Ramp / Mercury. If `brainstorming` already named a benchmark in the handoff,
   use that one — don't re-pick.
2. **Optional current-year search** — if web access is available, run 1–2 quick
   searches with the **current year** (e.g. "fintech landing page design 2026",
   `site:awwwards.com <vertical>`, or the brand's own site) to see what great
   looks like right now and sharpen the CTA/component target. If offline or
   search fails, the named brands in WEB-BENCHMARKS.md are enough — never block.
3. **Name what you'll adapt as a PATTERN, not an asset** — pin the specific
   component/CTA feel you'll match: "pill CTA with a soft shadow + hover-lift à
   la Stripe", "trust-row of logos like Xendit", "bento feature grid like
   Linear". Re-skin every one of these to the user's palette & content.

**Pattern, not property (hard rule — doctrine §2b):** you may adapt design
language, component shape, CTA style, spacing & motion. You may NOT copy or ship
the brand's **logo, name, wordmark, copy/headlines, photos, illustrations, or any
proprietary asset**. "CTA that feels like Gojek's" = good; a Gojek clone = not
ok. The concrete classes/keyframes still come from the local DNA; the web
benchmark only lifts the polish.

### 1.7. Proof-of-extraction gate (doctrine §7) — BEFORE writing any UI
Output a short **"Design DNA I'm using"** block — per chosen slug, list the
concrete values you just read (hero layout, headline classes, CTA classes,
section rhythm, the `@keyframes` names, the signature move from
`composition_techniques`, icon set) **plus the 1–2 web benchmark(s) from Step 1.6
and the exact pattern you're adapting from each** (e.g. "Xendit — trust logo-row
+ high-contrast CTA, adapted to our palette"). This proves the extraction is
real, not claimed. The values you show here MUST be the values in the code you
then write. Present it as part of your build plan; it doesn't need separate
approval, but it must be visible before Step 4.

### 2. If multi-pick: plan the mix
- **First pick = base layout.** Page structure & section ordering follow this.
- **Other picks = style sources.** Take the specific elements the user
  mentioned (e.g. "colors from #4", "buttons from #2") or, if they just said
  "combine", take the most prominent palette/accent/component from the extra
  references and apply them to the base.
- Don't glue two layouts into one messy result — the base stays the single
  backbone, the other references just "season" it.

### 3. Separate STYLE from CONTENT — MANDATORY
From the template, take **ONLY**:
- Visual style: colors, spacing, radius, shadow, typography, gradients.
- **Motion**: the actual `@keyframes`, transitions, and hover/scroll behaviors
  (doctrine §2 — carry these over, don't flatten the design into a static clone).
- Component structure & shape: layout grid, card/button/input shapes, the type
  & order of sections (hero, features, pricing, footer, etc.).
- **Icons** replace any emoji the template (or your instinct) would use — Lucide
  or Solar per doctrine §1. Never emit an emoji.

**DON'T** take / don't let leak into the output:
- Headlines, body copy, product descriptions, feature names, testimonials, or
  any business-specific text from the template.
- Brand names, logos, photos, original illustrations from the template.

Content rules:
- If the user **has** given content (product name, tagline, etc.) → use exactly
  that.
- If the user **hasn't** given content for a section → fill in a clearly-marked
  placeholder, e.g. `[Product headline here]`, `[Feature 1 description]`. Do NOT
  quietly use the template's original text as the "content".

If you find your output contains sentences nearly identical to the template's
text — that's a sign you're copying content, not style. Swap it for a
placeholder or ask the user.

### 3.5. Inspect the project structure BEFORE writing (doctrine §9) — don't guess where files go
Before you create a single file, look at the user's actual working directory and
write into the way it's really organized — never invent a folder or naming
convention from habit.

1. **List the working directory** and read its shape. Look for the tells:
   `package.json` (read it — framework, scripts, versions), `src/`, `app/`,
   `components/`, `pages/`, `public/`, `index.html`, `vite.config.*`,
   `next.config.*`, `tailwind.config.*`, `tsconfig.json`.
2. **Decide: existing project or empty folder?**
   - **Existing project** → slot into it. Put components where sibling components
     already live (`src/components/` vs `app/components/`), match the file naming
     (`PascalCase.tsx` / `kebab-case` / `index.html`), extend the styling system
     already in use (its Tailwind config / CSS convention), and reuse its import
     aliases (`@/components/...`). Do NOT create a parallel tree, rename the
     user's files, or reorganize their folders.
   - **Empty / bare folder (or just an `index.html`)** → THEN scaffold the
     standard structure for the target format below (Step 4).
3. **If it's ambiguous** (two plausible component folders, unclear framework,
   mixed codebase) → ask one short question (doctrine §4) before writing, so you
   don't drop files in the wrong place.

Carry the confirmed target path(s) into your plan/proof block so the user sees
exactly where each file will be written.

### 4. Convert to the target format

**HTML (default)**
- A single standalone `.html` file using **Tailwind via CDN**
  (`<script src="https://cdn.tailwindcss.com"></script>`), zero-setup — just
  open it in a browser.
- Include fonts inline (Google Fonts `<link>`) matching `fonts` in the index.
- **Icons**: add the chosen set's CDN and use icon elements, NOT emoji — Lucide
  `<script src="https://unpkg.com/lucide@latest"></script>` +
  `<i data-lucide="...">` + `lucide.createIcons()`, or Solar via the Iconify
  CDN (`<iconify-icon icon="solar:...">`). See doctrine §1 for exact snippets.
- **Animations**: paste the reference's `@keyframes` into a `<style>` block (or
  Tailwind config) and wire the same triggers — don't ship it static.
- **shadcn**: no install in plain HTML — replicate shadcn's patterns in Tailwind
  (radius, focus ring, padding scale) per doctrine §3 for any dialogs/tabs/etc.
- Save as `index.html` (or the name the user asks for) in the user's working
  directory.

**React (if requested)**
- **React 19** + **Vite 6** + **TypeScript** + **Tailwind v4**.
- Split into one component per section (e.g. `Hero.tsx`, `Features.tsx`),
  composed in `App.tsx`. Tailwind v4 via `@tailwindcss/vite`.
- **Icons**: `lucide-react` (default) or `@solar-icons/react` (premium/soft) —
  import components, never emoji (doctrine §1).
- **shadcn/ui**: for standard interactive components (dialog, dropdown, tabs,
  accordion, tooltip, form controls) use real shadcn — tell the user to run
  `npx shadcn@latest init` then `npx shadcn@latest add <components>` — then
  re-skin them to the reference palette so they aren't stock (doctrine §3).
- **Animations**: carry the reference's motion over (Tailwind keyframes in the
  CSS, or a small `@keyframes` block) — keep the premium feel.
- Include the minimal supporting files so it runs immediately: `package.json`
  (latest versions), `vite.config.ts`, `index.html`, `src/main.tsx`,
  `src/index.css` (with `@import "tailwindcss";`).
- Tell the user how to run it: `npm install && npm run dev`.

**Next.js (if requested)**
- **Next.js 15** (App Router, Server Components by default) + **TypeScript** +
  **Tailwind v4**.
- `app/` structure: `app/layout.tsx`, `app/page.tsx`, one component per section
  in `app/components/` or `components/`.
- **Icons**: `lucide-react` or `@solar-icons/react`, never emoji (doctrine §1).
- **shadcn/ui**: same as React — real shadcn for interactive components
  (`npx shadcn@latest init` / `add`), re-skinned to the palette (doctrine §3).
  shadcn works with the App Router; mark interactive ones `"use client"`.
- **Animations**: carry the reference's motion over; keep it feeling premium.
- Include `package.json`, `next.config.ts`, `tsconfig.json`, `app/globals.css`
  (Tailwind v4). Use Client Components (`"use client"`) only for parts that
  need interactivity.
- Tell the user: `npm install && npm run dev`.

> Framework versions: **always the latest for 2026** (React 19, Next.js 15,
> Tailwind v4). Don't downgrade without a reason.

### 5. Write the file & report (human language — doctrine §4)
- Run the doctrine §8 self-check first (emoji-free? DNA shown? hard floors §6 all
  pass — `py-20`+ sections, `text-5xl`+ hero, ≥2 real motions? refs named? icons
  in? shadcn where needed?).
- Write the output file(s) to the user's working directory (NOT into the bundled
  library folder `${CLAUDE_PLUGIN_ROOT}/references/` — that's read-only for us).
- Report like a collaborator, not a machine:
  - A 1–3 sentence plain-language summary of what you built and the direction.
  - **Name what you pulled and from where**, concretely — e.g. "hero + spacing
    from `aura-ai-landing`, the glow-on-hover CTA (`@keyframes pulse-glow`) from
    `signalis-saas-45`, Lucide icons". This is the proof it's reference-grounded.
  - The path(s) created + how to preview (HTML → open in a browser; React/Next →
    `npm install && npm run dev`).
  - Any unfilled placeholders the user needs to fill.
  - End with a next-step question / options — e.g. "Want me to add a pricing
    section, swap to Solar icons, or tweak the palette?" No dead-stop dump, no
    emoji.

## What this skill must NOT do
- **Invent the project's file structure (§9)** — never guess where files go. List
  the working directory first; slot into the existing folders/naming/styling
  convention, and only scaffold fresh when the folder is genuinely empty. Don't
  reorganize or rename the user's existing files.
- **Skip the DNA file** and build from memory — read `references/dna/<slug>.json`
  first and show the §7 proof block before writing UI.
- **Build the generic-AI layout (§6a)** — no centered-hero-of-doom + 3-card +
  purple-blob by reflex. Reproduce the reference's real `layout.hero_layout` and
  land ≥1 signature move; if you're centering a hero, the reference must be
  `centered`.
- **Ship under the hard floors (§6)** — no `text-3xl` heroes, no cramped
  sub-`py-20` sections, no fully-static page. Meet the numbers or say why not.
- **Emit any emoji** in the UI or the report — use Lucide/Solar icons (§1).
- **Invent from scratch** what a reference could ground — if you can't name the
  slug + the thing you pulled for a section, you're guessing (§2).
- **Drop the reference's motion/animation** and ship a static clone — paste the
  DNA's `@keyframes` and wire the triggers over (§2, §6).
- **Copy a web benchmark's brand assets (§2b)** — never ship a famous brand's
  logo, name, wordmark, copy, photos, or illustrations. Adapt the CTA/component
  *pattern* only, re-skinned to the user's palette; the concrete classes still
  come from the local DNA, not the brand.
- Copy / carry over the template's original content text into the output
  (headlines, copy, feature names) — the template is only a source of style,
  structure & motion.
- Fill an empty section with the template's original text as a "placeholder" —
  use a clearly-marked placeholder or ask the user.
- Reproduce brand assets exactly (logos, photos, illustrations) from the
  template.
- Downgrade framework versions without a reason (default is always latest).
- Rebuild a standard interactive component from zero when shadcn/ui defines a
  good one (React/Next) — use it and re-skin it (§3).
- Write output into the bundled library folder `${CLAUDE_PLUGIN_ROOT}/references/`.
- Build before the user has chosen via `brainstorming`.
- Dump files with no human summary / no next-step question (§4).
