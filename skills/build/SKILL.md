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
follow it. It governs the six things this skill depends on:
- **§0 reasoning pre-flight** — do it before you generate a single line.
- **§1 icons** — NO emoji, ever; use Lucide (tech/clean) or Solar (premium/soft).
- **§2 reference extraction** — you MUST open the reference HTML and physically
  lift its class strings, component shapes, AND animations/keyframes. Inventing
  from scratch is the failure mode this whole plugin exists to prevent.
- **§3 shadcn/ui** — use real shadcn for React/Next interactive components,
  its patterns for HTML, always re-skinned to the reference palette.
- **§4 communication** — human summary + options + a next-step question.
- **§6 self-check** — run it before you write files.

The steps below are the *what*; the doctrine is the *how*. Both are required.

Prerequisite: the user already chose candidates via the `brainstorming` skill.
If not, go back to `brainstorming` first.

## Input carried over from `brainstorming`
- List of chosen slugs, **in order** (first pick = base, the rest = style
  sources).
- Output format: `html` (default) / `react` / `next`.
- Any content context from the user (e.g. product name, button text).

## Steps

### 0. Reasoning pre-flight (doctrine §0) — do NOT skip
Before reading files or writing anything, run the six-step pre-flight from the
doctrine and show the user a short version:
1. Restate the goal in one sentence.
2. Name the exact reference slug(s) + the specific thing you'll pull from each.
3. List the sections/components you'll build, top to bottom.
4. Pick the icon set (Lucide or Solar) + one-line reason.
5. Note any component that needs a shadcn/ui pattern.
6. Name the biggest AI-slop risk and how you'll avoid it.

This is what makes Sonnet's output look designed instead of templated. If the
brief is missing something you need (format, mood, content), ask now (doctrine
§4) — don't guess.

### 1. Read the chosen template(s) — and EXTRACT, don't skim
For each slug, read its HTML file from
`${CLAUDE_PLUGIN_ROOT}/references/templates/<slug>.html`. Also read its entry in
`${CLAUDE_PLUGIN_ROOT}/references/index.json` (to know `colors`, `fonts`,
`sections`).

You are not reading for vibes — you are **extracting concrete material** (doctrine
§2). As you read, physically pull out:
- The **exact Tailwind class strings** for the hero, nav, cards, buttons,
  section wrappers (spacing scale, max-width, radius, shadow, gradient).
- The **component shapes** — how a card/button/pricing tier is structured.
- The **animations** — grep the file for `@keyframes`, `animate-`, `transition`,
  `transform`, `hover:`, `group-hover:`, and any scroll/intersection observer
  script. Copy the keyframes + their triggers into your output (adapt the
  colors). Motion is the #1 thing that makes a reference feel premium and the #1
  thing weak builds drop — do not drop it.

If an HTML file is missing → tell the user, offer to go back to `brainstorming`
to choose another. Don't proceed on a guess.

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
- Run the doctrine §6 self-check first (emoji-free? refs named? motion carried?
  icons in? shadcn where needed?).
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
- **Emit any emoji** in the UI or the report — use Lucide/Solar icons (§1).
- **Invent from scratch** what a reference could ground — if you can't name the
  slug + the thing you pulled for a section, you're guessing (§2).
- **Drop the reference's motion/animation** and ship a static clone — carry the
  `@keyframes` and triggers over (§2).
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
