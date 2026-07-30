---
name: glowup
description: Glow up (improve) an existing UI file by applying design fixes, pulling taste from the getokui library. Invoked after the review skill once the user approves fixes, OR directly when the user asks getokui to improve/polish/"perbaiki"/"glow up" a UI or component they already have — e.g. "getokui, glowup my landing page", "getokui perbaiki komponen ini", "getokui polish index.html". This skill edits the user's EXISTING file in place — upgrading spacing, color, typography, hierarchy & component polish using the curated references as the taste source (STYLE not content) — then reports what changed. For making something brand new, use brainstorming + build instead.
---

# getokui glowup — Improve an Existing UI Using the Taste Library

This skill's job: take a UI file the user **already has** and make it visibly
better — better spacing, color, typography, hierarchy, and component polish —
using getokui's curated references as the taste source. It **edits the real
file** (or writes an improved copy if the user prefers), then reports the
changes.

This is the second half of the **improve-what-exists** flow (`review` →
`glowup`). It can also run on its own if the user just says "glow this up"
without a prior review — in that case do a quick internal review first, then
apply.

## Prerequisite

The taste library is bundled at `${CLAUDE_PLUGIN_ROOT}/references/`. Sanity-check
that `${CLAUDE_PLUGIN_ROOT}/references/index.json` exists:
- **Present** → continue.
- **Missing** (unexpected — broken install) → tell the user and suggest
  reinstalling the plugin (`/plugin install`). Don't proceed on a guess.

## Input carried over from `review` (if any)
- The **target file path**.
- The **approved findings** (all, or the specific ones the user picked).
- If there was no `review` first: just the file path + the user's ask.

## Steps

### 1. Read the target file
Read the file the user pointed at. Understand its structure, framework
(plain HTML / React / Next), and current styling approach (Tailwind classes,
CSS, inline styles). Preserve that stack — a glowup improves the design, it does
NOT rewrite the file into a different framework unless the user asks.

If the file is missing/unreadable → tell the user, ask for the correct path.
Don't invent a file.

### 2. Decide the fixes
- **If findings came from `review`** → apply exactly those (the approved set).
- **If glowup was called directly** → do a quick internal pass over the same
  dimensions `review` uses (spacing, color, typography, hierarchy, component
  polish, consistency) and pick the highest-impact improvements. Briefly tell
  the user what you're about to change before doing it if the scope is large.

### 3. Pull taste from the library — STYLE, not content — MANDATORY
Pick 1–3 relevant references from `index.json` (same vertical/mood as the
target). Read their HTML/`colors`/`fonts` as needed. From them, borrow **only**:
- Visual style: color palette, spacing scale, radius, shadow, typography,
  transitions.
- Component structure & shape: card/button/input shapes, section rhythm, grid.

**DON'T** copy the reference's **content** into the user's file:
- No headlines, body copy, product names, testimonials, feature text from the
  template.
- No brand names, logos, photos, illustrations from the template.
- **Keep the user's own content intact** — glowup restyles what's already there.
  Never replace the user's real text/data with a template's text. If a section
  is empty, use a clearly-marked placeholder (e.g. `[Feature title]`), never the
  template's original words.

### 4. Apply the improvements
Edit the file in place, keeping its framework and its content:
- Adjust spacing to a consistent scale, fix contrast, strengthen the type
  hierarchy, unify radius/shadow/border tokens, tidy the layout/grid, polish
  component states.
- Make **surgical, coherent** edits — don't gut the file. The result should be
  recognizably the user's UI, leveled up.
- Match the existing tooling: Tailwind classes if it uses Tailwind, its CSS
  convention otherwise. Don't introduce a new styling system unasked.
- If the user asked for a non-destructive result, write an improved copy
  (e.g. `index.glowup.html`) instead of overwriting — otherwise edit in place.

Write output to the **user's working directory / their file** — NEVER into the
bundled library `${CLAUDE_PLUGIN_ROOT}/references/` (read-only for us).

### 5. Report what changed
Tell the user concisely:
- The file path that was updated (or the new copy's path).
- A short bullet list of the improvements made, grouped by dimension
  (spacing / color / typography / hierarchy / polish), each noting which
  reference inspired it where relevant.
- Anything left for the user (e.g. placeholders to fill, an image to swap).
- How to preview: HTML → "open it in a browser"; React/Next → "run `npm run
  dev`".

## What this skill must NOT do
- Replace the user's real content with the template's text/copy — restyle only,
  keep their content.
- Reproduce brand assets (logos, photos, illustrations) from the reference.
- Rewrite the file into a different framework unless the user explicitly asks.
- Gut/replace the whole UI — glowup is an upgrade, not a from-scratch rebuild
  (that's `brainstorming` + `build`).
- Write into the bundled library folder `${CLAUDE_PLUGIN_ROOT}/references/`.
