---
name: build
description: Adapt the getokui reference(s) the user picked into the user's own UI, in the requested format (HTML default, React 19, or Next.js 15). Invoked after the pick skill once the user has chosen one or more candidates. This skill reads the chosen template file(s) from the local library, TAKES the visual style & structure (NOT the template's original text content), mixes when there are several picks (first pick = base layout, the rest = style sources), converts to the target format, then writes the output file. Do not invoke this skill before the user has chosen via pick.
---

# getokui build — Adapt into the User's Own UI

This skill's job: turn the chosen reference(s) into real UI code that belongs to
the user — not a raw copy. Take the **style & structure**, fill with the
**user's content** (or clear placeholders), then emit in the requested format.

Prerequisite: the user already chose candidates via the `pick` skill. If not,
go back to `pick` first.

## Input carried over from `pick`
- List of chosen slugs, **in order** (first pick = base, the rest = style
  sources).
- Output format: `html` (default) / `react` / `next`.
- Any content context from the user (e.g. product name, button text).

## Steps

### 1. Read the chosen template(s)
For each slug, read its HTML file from
`~/.getokui/references/templates/<slug>.html`. Also read its entry in
`index.json` (to know `colors`, `fonts`, `sections`).

If an HTML file is missing → tell the user, offer to go back to `pick` to choose
another. Don't proceed on a guess.

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
- Visual style: colors, spacing, radius, shadow, typography, motion/transitions.
- Component structure & shape: layout grid, card/button/input shapes, the type
  & order of sections (hero, features, pricing, footer, etc.).

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
- Save as `index.html` (or the name the user asks for) in the user's working
  directory.

**React (if requested)**
- **React 19** + **Vite 6** + **TypeScript** + **Tailwind v4**.
- Split into one component per section (e.g. `Hero.tsx`, `Features.tsx`),
  composed in `App.tsx`. Tailwind v4 via `@tailwindcss/vite`.
- Include the minimal supporting files so it runs immediately: `package.json`
  (latest versions), `vite.config.ts`, `index.html`, `src/main.tsx`,
  `src/index.css` (with `@import "tailwindcss";`).
- Tell the user how to run it: `npm install && npm run dev`.

**Next.js (if requested)**
- **Next.js 15** (App Router, Server Components by default) + **TypeScript** +
  **Tailwind v4**.
- `app/` structure: `app/layout.tsx`, `app/page.tsx`, one component per section
  in `app/components/` or `components/`.
- Include `package.json`, `next.config.ts`, `tsconfig.json`, `app/globals.css`
  (Tailwind v4). Use Client Components (`"use client"`) only for parts that
  need interactivity.
- Tell the user: `npm install && npm run dev`.

> Framework versions: **always the latest for 2026** (React 19, Next.js 15,
> Tailwind v4). Don't downgrade without a reason.

### 5. Write the file & report
- Write the output file(s) to the user's working directory (NOT into the library
  folder `~/.getokui/references/` — that's read-only for us).
- Report to the user: the path(s) created, which reference is the base + style
  sources, and how to preview:
  - HTML → "open `index.html` in a browser".
  - React/Next → "run `npm install && npm run dev`".
- If there are unfilled placeholders, list which ones so the user knows what to
  fill in.

## What this skill must NOT do
- Copy / carry over the template's original content text into the output
  (headlines, copy, feature names) — the template is only a source of style &
  structure.
- Fill an empty section with the template's original text as a "placeholder" —
  use a clearly-marked placeholder or ask the user.
- Reproduce brand assets exactly (logos, photos, illustrations) from the
  template.
- Downgrade framework versions without a reason (default is always latest).
- Write output into the library folder `~/.getokui/references/`.
- Build before the user has chosen via `pick`.
