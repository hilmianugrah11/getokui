---
name: review
description: Review an EXISTING UI file's design quality against the getokui taste library (213 curated landing pages), and STOP. Invoked when the user asks getokui to critique/review/audit/"cek design" of a UI they already have — e.g. "getokui, review my landing page", "getokui review index.html", "getokui, cek design komponen ini". This skill reads the file the user points at, compares it against the curated references in index.json (spacing, color, typography, hierarchy, layout, consistency), then presents RANKED design findings and STOPS — it does NOT edit or rewrite the file (that's the glowup skill). Use this for "improve what exists"; for making something new, use the brainstorming skill instead.
---

# getokui review — Critique an Existing UI Against the Taste Library

This skill's job: look at a UI file the user **already has**, judge its design
quality **relative to getokui's curated references**, and hand back a ranked,
actionable list of what's weak — then **stop**. It does not touch the file.
Applying the fixes is the `glowup` skill's job, and only after the user says go.

This is the **improve-what-exists** entry point. (To create a new UI from
scratch, use `brainstorming` → `build`.)

## READ FIRST — shared doctrine (mandatory)

Read `${CLAUDE_PLUGIN_ROOT}/skills/shared/DOCTRINE.md` and follow it. For review
the load-bearing parts are:
- **§0 reasoning pre-flight** — know the target's vertical/mood + which refs you
  measure against before you critique.
- **§1 icons** — flag emoji-in-UI as a finding (fix: swap to Lucide/Solar).
- **§2 reference extraction** — anchor findings to what a real reference does.
  You can read a relevant reference's Design DNA
  (`references/dna/<slug>.json`) to cite its exact spacing/motion in a finding,
  e.g. "hero is `text-3xl`; `novapay` runs `text-6xl lg:text-7xl`".
- **§3 shadcn/ui** — a valid finding can be "this hand-rolled dropdown should be
  a shadcn component" (for React/Next targets).
- **§4 communication** — human summary + ranked findings + a real question; NO
  emoji.
- **§5 checkpoint** — HARD STOP after findings; do not edit.
- **§6 hard floors** — measure against the concrete minimums: a sub-`text-5xl`
  hero, cramped sub-`py-20` sections, a fully-static page (no real motion), or
  emoji-as-icons are each concrete, citable findings — not matters of taste.

## Prerequisite

The taste library is bundled at `${CLAUDE_PLUGIN_ROOT}/references/`. Sanity-check
that `${CLAUDE_PLUGIN_ROOT}/references/index.json` exists:
- **Present** → continue.
- **Missing** (unexpected — broken install) → tell the user and suggest
  reinstalling the plugin (`/plugin install`). Don't proceed on a guess.

## Steps

### 1. Identify the target file
The user points at a file — a path (e.g. `index.html`, `src/components/Hero.tsx`,
`app/page.tsx`). Determine which file(s) to review.
- If the path is **clear** → read it.
- If the user said "review my UI" with **no path** and it's ambiguous → ask one
  short question: "which file should I review? (give me the path)". Don't guess
  across the whole project.
- Read the file. If it's huge, focus on the markup/JSX + styling (Tailwind
  classes, CSS, style tokens) — that's what design review needs.

**Honest scope:** you review the **code** (structure, Tailwind classes, spacing
scale, color tokens, typography, component shapes), not a rendered pixel
screenshot — the plugin has no renderer. Say so if the user expects a visual
diff. Code-level review is still the bulk of design quality.

### 2. Pick comparison references from the library
Read `${CLAUDE_PLUGIN_ROOT}/references/index.json`. Infer what kind of UI the
target is (a landing hero? a pricing section? a fintech page?) and pick **1–3
relevant references** to measure against — same vertical/mood where possible
(e.g. a fintech landing → compare against `sequra-fintech`, `novapay`). If you
need detail, read those templates' HTML from
`${CLAUDE_PLUGIN_ROOT}/references/templates/<slug>.html` and their `colors` /
`fonts` from the index.

This is what makes a getokui review different from generic advice: findings are
anchored to **real, curated examples**, e.g. "your card padding is `p-2`; the
reference `obsidian` breathes at `p-6`/`p-8` — tighten the rhythm."

### 3. Evaluate across design dimensions
Go through the target against these dimensions, comparing to the reference(s):
- **Spacing & rhythm** — padding/margin scale, whitespace, section breathing,
  alignment to a consistent scale.
- **Color & contrast** — palette cohesion, accent usage, text/background
  contrast (readability), dark/light consistency.
- **Typography** — font pairing, size hierarchy (h1→body), weight/leading,
  line-length.
- **Layout & hierarchy** — grid, visual hierarchy, focal point, the order and
  prominence of sections, responsive shape.
- **Component polish** — radius/shadow/border consistency, button & input
  states, card shapes.
- **Iconography** — emoji used where an icon belongs (flag it: swap to
  Lucide/Solar per doctrine §1); inconsistent or default-sized icons.
- **Motion** — is the UI static where a reference would have tasteful animation
  (hover states, entrance, gradient shift)? Naming the missing motion + the ref
  that does it well is a high-value finding (doctrine §2).
- **Interactive components** — hand-rolled dialogs/tabs/dropdowns that shadcn/ui
  would do better (React/Next targets — doctrine §3).
- **Consistency** — repeated tokens vs one-off values, spacing/colors that drift.

For each real issue, note: **where** (line or element), **what's wrong**, **why
it matters**, and **which reference does it better** (name it).

### 4. Present RANKED findings — then HARD STOP
Present a short, readable review in plain human language (doctrine §4), no emoji:
- A 1–2 sentence overall impression (honest — if it's already good, say so).
- Findings **ranked most-impactful first**, each as:
  `element/line — what's weak → the fix (ref: <slug> does this well)`.
- Keep it to the **top ~5–8** findings that actually move the needle; don't pad
  with nitpicks. If the UI is genuinely solid, say "these are minor" rather than
  inventing problems.

Close with an explicit handoff invitation:
> "Want me to glow this up? I can apply these fixes (say 'glowup' or pick which
> findings), or you can tweak them yourself."

**HARD STOP here.** Do NOT edit the file, do NOT auto-run `glowup`. Wait for the
user to decide which fixes to apply (or all, or none) in the next message. This
is getokui's checkpoint — the agent critiques, the user decides.

### 5. Interpret the user's answer (next turn)
- **"glowup" / "apply all"** → hand off to the `glowup` skill with the full
  findings list + the target file path.
- **"apply #1 and #3"** → hand off to `glowup` with just those findings.
- **"none / I'll do it"** → stop gracefully; offer to review again later.

## What this skill must NOT do
- Edit, rewrite, or create files — that's `glowup`'s job. Review only reads.
- Auto-continue to `glowup` without the user choosing (the hard stop is
  mandatory).
- Invent problems to look thorough — if the UI is good, be honest.
- Give generic advice with no anchor — tie findings to the library references.
- Claim a pixel/visual render — be clear it's a code-level design review.
