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

## READ FIRST — shared doctrine (mandatory)

Read `${CLAUDE_PLUGIN_ROOT}/skills/shared/DOCTRINE.md` and follow it. For glowup
the load-bearing parts are:
- **§0 reasoning pre-flight** — decide refs + what you'll pull before editing.
- **§1 icons** — replace ANY emoji you find (or would add) with Lucide/Solar
  icons; never leave or introduce emoji.
- **§2 reference extraction** — read the pre-extracted **Design DNA file**
  (`references/dna/<slug>.json`) FIRST for the real class strings, spacing, and
  `@keyframes`, then open the HTML for anything it misses. A glowup that adds the
  DNA's motion is far stronger than one that only nudges spacing.
- **§3 shadcn/ui** — for weak interactive components, upgrade to shadcn
  (React/Next) or its pattern (HTML), re-skinned to the palette.
- **§4 communication** — human summary + options + a next-step question.
- **§6 hard floors + §6a anti-slop** — bring the file UP to the minimums:
  `py-20`+ sections, `text-5xl`+ hero, ≥2 real motions, 0 emoji, one radius +
  shadow; and if it's the generic centered-hero/3-card slop, push it toward the
  reference's real composition + a signature move (don't just re-space the slop).
- **§7 proof-of-extraction gate** — show the "Design DNA I'm using" block (the
  tokens you're pulling in) alongside the fixes, before you edit.
- **§8 self-check** — run it before you save.

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

### 3. Pull taste from the library — DNA first, STYLE not content — MANDATORY
Pick 1–3 relevant references from `index.json` (same vertical/mood as the
target). Then, for each:
1. **Read its Design DNA** `${CLAUDE_PLUGIN_ROOT}/references/dna/<slug>.json`
   FIRST (doctrine §2 "Start from the pre-extracted DNA") — it gives you the real
   `hero.h1_classes`, `hero.cta_classes`, `spacing.*`, `radius`, `shadow`,
   `layout.hero_layout` + `layout.composition_techniques` (the composition +
   signature move — doctrine §6a anti-slop), and `motion.keyframes_css` without
   guessing.
2. **Open the HTML** `${CLAUDE_PLUGIN_ROOT}/references/templates/<slug>.html` only
   for what the DNA doesn't capture (fuller component structure, the observer
   script). Don't glow up from memory.

Then show the **proof-of-extraction block** (doctrine §7) — the DNA tokens you're
about to pull in — alongside the fixes, before you edit. Borrow **only**:
- Visual style: color palette, spacing scale, radius, shadow, typography — reuse
  the DNA's real Tailwind class strings, not made-up round numbers.
- **Motion**: paste the DNA's `motion.keyframes_css` verbatim and wire the
  triggers (`hover:`, `group-hover:`, scroll observers) into the user's file
  (adapt colors). This is often the single biggest glow-up lever.
- Component structure & shape: card/button/input shapes, section rhythm, grid.
- **Icons**: if the user's file uses emoji, replace them with Lucide/Solar icons
  (doctrine §1) as part of the glow-up.

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
- **Add the motion you extracted** (doctrine §2) — paste the reference's
  `@keyframes`/transitions and wire the triggers so the UI feels alive, not
  static. For HTML add the icon-set CDN + `lucide.createIcons()` if you swapped
  in icons; for React/Next add the icon import.
- **Replace every emoji** with a Lucide/Solar icon, sized and colored to the
  palette (doctrine §1).
- For weak interactive components (a hand-rolled dialog/tabs/dropdown), upgrade
  to shadcn/ui (React/Next) or its pattern in Tailwind (HTML), re-skinned to the
  palette (doctrine §3) — only if the user's stack allows it without a big rewrite.
- Make **surgical, coherent** edits — don't gut the file. The result should be
  recognizably the user's UI, leveled up.
- Match the existing tooling: Tailwind classes if it uses Tailwind, its CSS
  convention otherwise. Don't introduce a new styling system unasked.
- If the user asked for a non-destructive result, write an improved copy
  (e.g. `index.glowup.html`) instead of overwriting — otherwise edit in place.

Write output to the **user's working directory / their file** — NEVER into the
bundled library `${CLAUDE_PLUGIN_ROOT}/references/` (read-only for us).

### 5. Report what changed (human language — doctrine §4)
Run the doctrine §8 self-check (including the §6 hard floors — did the glowup
bring the file up to `py-20`+ sections, `text-5xl`+ hero, ≥2 real motions, 0
emoji, one radius + shadow token?), then tell the user like a collaborator:
- A 1–2 sentence plain-language summary of how the UI leveled up.
- The file path that was updated (or the new copy's path).
- A short bullet list of the improvements, grouped by dimension
  (spacing / color / typography / hierarchy / motion / polish), each **naming
  the reference** it came from where relevant (e.g. "added the card hover-lift
  from `obsidian`", "swapped emoji for Lucide icons").
- Anything left for the user (e.g. placeholders to fill, an image to swap).
- End with a next-step question / options — e.g. "Want me to push the motion
  further, or glow up another section?" No emoji, no dead-stop dump.

## What this skill must NOT do
- **Skip the DNA file** — read `references/dna/<slug>.json` first and show the §7
  proof block before editing; don't glow up from memory.
- **Leave the file under the hard floors (§6)** — if the hero is `text-3xl` or a
  section is cramped or the page is static, the glowup must lift it to the floor.
- **Leave or add emoji** — replace them with Lucide/Solar icons (§1).
- **Glow up from memory** — use the DNA's real classes + `keyframes_css`; don't
  just nudge numbers you guessed (§2).
- Replace the user's real content with the template's text/copy — restyle only,
  keep their content.
- Reproduce brand assets (logos, photos, illustrations) from the reference.
- Rewrite the file into a different framework unless the user explicitly asks.
- Gut/replace the whole UI — glowup is an upgrade, not a from-scratch rebuild
  (that's `brainstorming` + `build`).
- Write into the bundled library folder `${CLAUDE_PLUGIN_ROOT}/references/`.
- Report as a bare file dump with no human summary / next-step question (§4).
