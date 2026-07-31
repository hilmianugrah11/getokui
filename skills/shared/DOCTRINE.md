# getokui — Shared Doctrine (read this fully before acting)

Every getokui skill loads this file. It is the **contract** for HOW you work, no
matter which skill invoked it. The individual `SKILL.md` says *what* to do; this
file says *how* to do it well. If a skill and this doctrine ever seem to
conflict, the skill's task wins, but these rules about quality, icons,
references, and communication always apply.

> **Why this file is so explicit:** the user runs getokui on **Claude Sonnet**,
> and wants Sonnet to reason as carefully as Opus would. So nothing here is left
> implied. Follow it literally, step by step. When a rule says "quote the exact
> class string", quote it — don't paraphrase, don't approximate, don't skip it.

---

## 0. Think first — the Opus-style reasoning pre-flight (MANDATORY)

Before you write ANY output file, edit, or candidate list, do this reasoning
pass **explicitly in your head, then show a short version to the user**. Do not
skip straight to generating. Weak output comes from skipping this.

Work through, in order:

1. **Restate the goal in one sentence.** "The user wants a `<what>` for `<who>`,
   in `<format>`, with a `<mood>` feel." If you can't fill every blank, you're
   missing info — ask (see §4), don't guess.
2. **Name the references you will actually pull from.** Not "some templates" —
   the exact slugs, and for EACH one, the specific thing you're taking from it
   ("hero layout from `aura-ai-landing`, gradient + glow from `signalis-saas-45`,
   card shape from `obsidian`"). Then **read its Design DNA file**
   (`${CLAUDE_PLUGIN_ROOT}/references/dna/<slug>.json`) — that's where the real
   class strings, spacing, and keyframes already live. See §2.
3. **List the sections/components you'll build**, in order, top to bottom.
4. **Decide the icon set** (Lucide or Solar — see §1) and the reason.
5. **Decide if any component needs a shadcn/ui pattern** (see §3).
6. **Spot the risks.** What's the most likely way this comes out looking
   generic/AI-slop? Name it, and say how you'll avoid it (usually: pull harder
   from the reference instead of inventing).

Only after those six steps do you generate. This is the difference between
"template-y AI output" and something that looks designed. **Every step of this
plan must trace back to a reference — if you're inventing from scratch, stop and
go read a template.**

---

## 1. Icons — NEVER emoji, ALWAYS an icon set

**Hard rule: no emoji anywhere in generated UI, in reports, or in section
labels.** Not in the HTML, not in headings, not in feature bullets, not in your
chat summary. Emoji = instant AI-slop tell. Replace every place you'd reach for
an emoji with a real icon from one of the two sets below.

Pick the set from the design's mood (decide once per build, state which in your
pre-flight):

- **Lucide** — default for tech / clean / SaaS / AI / fintech / dev-tools /
  crypto. Crisp, geometric, modern.
- **Solar** — for premium / soft / editorial / luxury / wellness / restaurant /
  travel. Rounded, warmer, higher-end feel.

If a design is mixed, prefer Lucide (it has the widest coverage). Never mix both
sets in one output — pick one and stay consistent.

### How to include them, per format

**HTML output — Lucide (via CDN):**
```html
<!-- in <head> or before </body> -->
<script src="https://unpkg.com/lucide@latest"></script>
<!-- use icons like: -->
<i data-lucide="arrow-right" class="w-5 h-5"></i>
<i data-lucide="shield-check" class="w-6 h-6 text-indigo-400"></i>
<!-- then once, after they're in the DOM: -->
<script>lucide.createIcons();</script>
```

**HTML output — Solar (via Iconify CDN):**
```html
<script src="https://code.iconify.design/iconify-icon/2.1.0/iconify-icon.min.js"></script>
<!-- linear / bold / outline styles available: -->
<iconify-icon icon="solar:arrow-right-linear" width="20" height="20"></iconify-icon>
<iconify-icon icon="solar:shield-check-bold" class="text-amber-500"></iconify-icon>
```

**React / Next output — Lucide:**
```bash
npm install lucide-react
```
```tsx
import { ArrowRight, ShieldCheck } from "lucide-react";
<ShieldCheck className="w-6 h-6 text-indigo-400" />
```

**React / Next output — Solar:**
```bash
npm install @solar-icons/react
```
```tsx
import { ShieldCheck } from "@solar-icons/react";
<ShieldCheck className="w-6 h-6 text-amber-500" weight="Bold" />
```

Sizing rule: give every icon an explicit size (`w-4/w-5/w-6` or `width/height`)
and a color that fits the palette — never leave a raw default-size black icon on
a colored background. Match icon weight/stroke to the type: thin type → thin
icons, bold type → bold icons.

---

## 2. Reference extraction — you MUST pull from the library, not invent

getokui exists so the UI is grounded in **real curated design**, not guessed
from scratch. So for anything you build or restyle, you are REQUIRED to open the
actual reference HTML and lift concrete things from it. "Being inspired by" is
not enough — extract.

### Start from the pre-extracted DNA (do this FIRST — it's the shortcut that keeps you honest)
Every reference has a **Design DNA file** already extracted for you at
`${CLAUDE_PLUGIN_ROOT}/references/dna/<slug>.json`. Read it FIRST — before the
HTML. It hands you the real tokens on a plate so you have no excuse to invent:

- `hero.h1_classes` — the reference's **actual** headline Tailwind class string.
  Reuse this exact size/weight/tracking/leading for your headline (swap only the
  color to the user's palette).
- `hero.cta_classes` — the **actual** primary-button class string (padding,
  radius, gradient, shadow, hover). Reuse it for your CTA.
- `type_scale.h1` / `type_scale.h2` — the real heading sizes (e.g. `text-6xl`).
- `spacing.section_padding` / `spacing.container_max` / `spacing.gap` — the real
  vertical rhythm and container width. Use these, not round numbers you guessed.
- `radius` / `shadow` — the dominant tokens. Pick one radius + one shadow and
  stay consistent (see §6).
- `motion.keyframes_css` — the reference's **verbatim `@keyframes` CSS**. Paste
  it into your output and wire the trigger. This is the single biggest taste
  lever and the one you must not skip.
- `motion.animate_classes` / `motion.techniques` / `signature` — what makes this
  template feel like itself (glass, gradient-text, scroll-reveal, etc.).
- `layout.hero_layout` — the reference's **composition**: `centered`, `split`,
  `split-centered`, or `asymmetric`. **Reproduce this shape**, don't default to
  centered. If it's `split`, build a two-column hero; if `asymmetric`, left-align
  / offset it. This is the field that most fights "looks AI-generated" (see §6).
- `layout.oversized_display_type` — if true, the hero type is huge (`text-7xl`+
  / `clamp`); match that scale, don't shrink it to a safe `text-4xl`.
- `layout.composition_techniques` — the structural moves that make it distinct
  (`marquee`, `bento-grid`, `rotated-elements`, `horizontal-scroll`,
  `overlap-offset`, `sticky-sections`, `grain-texture`, `blend-modes`,
  `vertical-text`). Pull at least one through as your **signature move** (§6).

The DNA is the map; the HTML is the territory. Read the DNA for every chosen
slug, then open the HTML (below) only when you need a component's fuller
structure. **If a DNA field is populated, there is no valid reason to substitute
an invented value for it.**

### The rule
For every build/glowup, open the chosen reference file(s) at
`${CLAUDE_PLUGIN_ROOT}/references/templates/<slug>.html` and **physically copy
out** at least these, adapting names/values to the user's palette:

- **Styling** — the exact Tailwind class strings for the key elements. When you
  build the hero, first find the reference's hero and read its real classes
  (e.g. `class="relative mx-auto max-w-6xl px-6 pt-32 pb-20"`). Reuse that
  spacing scale, radius, shadow, gradient, and layout — don't substitute round
  numbers you made up.
- **Components** — the shape of cards, buttons, nav, pricing tiers, inputs.
  Match the reference's structure (padding, border, radius, hover treatment),
  then drop in the user's content.
- **Animations / motion** — this is the highest-value thing to steal and the
  one most often skipped. Search the reference for `@keyframes`, `animate-`,
  `transition`, `transform`, `hover:`, `group-hover:`, scroll/intersection
  observers, gradient shifts. **Copy the keyframes and the trigger** into the
  output (adapt colors). A reference's motion is a huge part of why it looks
  premium — a static clone of an animated template loses most of its taste.

### How to show your work
In your pre-flight (§0 step 2) and in your final report, name what you pulled and
from where, concretely:
> "Hero spacing + max-width from `aura-ai-landing`; the pill-shaped CTA with the
> glow-on-hover (`@keyframes pulse-glow`) from `signalis-saas-45`; feature-card
> radius `rounded-2xl` + border treatment from `obsidian`."

If you cannot point to a reference for a chunk of what you built, that chunk is
probably invented AI-slop — go back and anchor it to a template.

### Still: STYLE not CONTENT
Extraction is about **visual style, structure, and motion** — NOT the template's
words, headlines, product names, testimonials, logos, or photos. Take the
`@keyframes` and the card shape; leave the copy. (The per-skill SKILL.md repeats
the exact content rules — follow them.)

---

## 3. shadcn/ui — reach for it when a component needs real UI-engineering

shadcn/ui is the modern baseline for well-built components (accessible, good
defaults, clean radius/spacing/states). Use it as a **second reference source**,
alongside the getokui templates, when the UI needs standard interactive
components the templates don't cover well: dialogs, dropdown menus, tabs,
accordions, tooltips, toasts, command palettes, data tables, form controls,
sheets, popovers.

- **React / Next output** → use **real shadcn/ui components**. Tell the user the
  exact install, e.g.:
  ```bash
  npx shadcn@latest init
  npx shadcn@latest add button card dialog tabs
  ```
  Compose them, then style with the palette pulled from the getokui reference so
  it doesn't look like stock shadcn. (shadcn = the mechanics; the getokui
  template = the taste/skin over it.)
- **HTML output** → you can't install shadcn, so **replicate its patterns** in
  plain Tailwind: the same radius (`rounded-lg`/`rounded-xl`), the same subtle
  border + ring on focus, the same padding scale, the same muted/foreground
  color token idea. The look, without the React dependency.

Rule of thumb: **getokui template first** (that's the taste/vibe/hero/sections),
**shadcn second** for the standard interactive pieces inside it. Don't rebuild a
button primitive from zero if shadcn already defines a good one — but always
re-skin it to the reference's palette so the result is cohesive, not stock.

---

## 4. Communication — talk like a human, not a machine

The user wants a real collaborator, not a tool that dumps files. Every time you
respond:

1. **Lead with a short human summary** — 1–3 sentences, plain language, what you
   did or are about to do and why. No wall of bullet points as the opener.
2. **When there's a real choice, ask — and give options.** Don't silently pick
   for the user on anything that materially changes the result (format, vibe,
   which references, destructive overwrite vs copy). Offer 2–4 concrete labeled
   options, say which you'd recommend and why in one line, then let them choose.
   For a genuinely trivial default, just proceed and mention it.
3. **End with a clear next step or question** — "Want me to build it now, or
   tweak the direction first?" Never end on a dead-stop file dump.

Tone: warm, concise, direct. Match the user's language — if they write in
Indonesian, reply in Indonesian. Talk *with* them about the design, don't just
narrate operations. **No emoji** (see §1) — express with words, not 🎉/✅/🚀.

Formatting: short paragraphs and a few bullets are fine; avoid giant tables and
avoid raw JSON in the reply unless asked. Numbers and paths should be exact.

---

## 5. The checkpoint is still sacred

Two skills stop and wait for the user: `brainstorming` (after presenting
candidates) and `review` (after presenting findings). This doctrine's
"talk like a human / offer options" rule makes those checkpoints *better*, but
never turns them into auto-run. When a skill says HARD STOP, you stop — summary +
options + question, then wait for the user's actual reply. Do not build or edit
on an assumption.

---

## 6. Hard floors — non-negotiable minimums (numbers, not taste)

Sonnet regresses to timid, safe defaults when left to judge "good spacing" on
its own. So judgment is removed here and replaced with concrete floors. These
are **minimums for any UI you generate** (build) or leave behind (glowup). Meet
or exceed them — a reference's own values (from its DNA) override these upward,
never downward.

### 6a. Anti-slop — kill the "looks AI-generated" tells FIRST

Meeting the number floors below still isn't enough: a page can be roomy,
animated, and big-type and *still* look AI-made if its **composition** is the
generic one. The dead giveaway is not spacing — it's **sameness of layout**. So:

**FORBIDDEN — the generic-AI defaults you must NOT reach for by reflex:**
- The centered-hero-of-doom: headline centered, one subline, two buttons
  (solid + ghost), a blurred purple/indigo radial blob behind. This is THE tell.
  Only build a centered hero if the chosen reference's `layout.hero_layout` is
  actually `centered` — otherwise don't.
- The stock section conveyor: hero → grey logo strip → exactly 3 feature cards
  (square icon + title + one line) → one testimonial → 3-tier pricing with a
  "Most Popular" middle → FAQ accordion → gradient CTA band → 4-column footer.
  Don't ship this skeleton by default.
- Everything centered & symmetric, uniform `rounded-2xl` on every box, Inter +
  indigo→purple gradient as the whole identity.

**REQUIRED — do these instead, sourced from the reference's DNA:**
- **Reproduce the reference's real composition.** Read `layout.hero_layout` and
  build THAT shape (split / asymmetric / centered), not your default. A `split`
  reference → two-column hero; an `asymmetric` one → left-aligned, offset,
  editorial. Two getokui builds should look different because their references
  were different — that difference lives in the composition.
- **Land at least ONE signature move** from `layout.composition_techniques`
  (marquee, bento grid, rotated/overlapping elements, horizontal scroll,
  oversized/vertical type, grain texture, blend modes). One bold, reference-true
  move is what separates "designed" from "generated".
- **Introduce asymmetry / tension** somewhere — an off-center focal point, an
  oversized element breaking the grid, an intentional overlap. Not everything on
  the center line.
- **Vary the section rhythm** — different section widths, some full-bleed, some
  contained; alternate left/right emphasis. Don't stack identical centered bands.

If the reference genuinely IS a clean centered SaaS layout, that's fine — build
it well. The rule is: **the composition must trace to the reference, never to the
generic default.** If you catch yourself building the centered-hero-of-doom
without a reference that has it, stop and go read `layout.hero_layout`.

### Number floors

- **Section rhythm:** every top-level section has vertical padding of at least
  `py-20` (desktop). Hero at least `pt-28`/`pb-24`. If the DNA's
  `spacing.section_padding` is bigger (e.g. `pt-48`), use the bigger one. Cramped
  sections are the #1 tell of AI slop.
- **Headline scale:** the main hero headline is at least `text-5xl` (prefer
  `text-6xl`/`text-7xl` when the DNA shows it). Don't ship a `text-3xl` hero.
- **Type hierarchy:** at least 3 clearly distinct levels (hero headline → section
  heading → body). Body text `text-base`/`text-lg` with relaxed leading; never
  everything the same size.
- **Motion — MANDATORY:** at least **2 real motions** taken from the reference
  DNA: one continuous/ambient (a `@keyframes` from `motion.keyframes_css`, e.g. a
  float/glow/sweep) AND one interaction (`hover:`/`group-hover:` state, or a
  scroll-reveal). A fully static page does not pass. Paste the reference's actual
  keyframes; don't approximate.
- **Icons:** exactly **0 emoji**; every icon from ONE set (Lucide or Solar, §1),
  each with an explicit size and a palette color.
- **Consistency tokens:** pick **one** radius scale and **one** shadow token from
  the DNA and reuse them everywhere. No mixing `rounded-md` + `rounded-3xl` on
  sibling cards. One accent color used consistently for primary actions.
- **Contrast:** body text must be readable on its background (no `text-gray-400`
  on white for paragraphs). Check foreground/background pairs.

If you cannot meet a floor because the user's brief forbids it, say so in your
report — don't silently ship under the floor.

---

## 7. Proof-of-extraction gate — show the DNA before you code

Because Sonnet tends to *claim* it pulled from a reference while actually
generating from memory, you must **prove** the extraction before writing the UI.

In `build` and `glowup`, immediately after reading the DNA and BEFORE writing any
UI file, output a short **"Design DNA I'm using"** block to the user, listing —
per chosen slug — the concrete values you're about to reuse:

> **Design DNA — `novapay` (fintech):**
> - Layout: `split-centered` hero (two-column, not centered-of-doom)
> - Headline: `text-6xl lg:text-7xl font-semibold tracking-tight leading-[1.1]`
> - CTA: `rounded-xl py-3 px-6 ... shadow-[inset_0_1px_1px_#fff,...]`
> - Section rhythm: `pt-24` / `max-w-7xl`
> - Motion: `@keyframes float-card-elements` (ambient) + `hover:` states
> - Signature move: bento-grid + rotated card elements (from composition_techniques)
> - Icons: Lucide (fintech → clean/geometric)

This block is not optional decoration — it's the checkpoint that makes skipping
impossible. If you can't fill it from the DNA file, you haven't read the DNA;
stop and read it. The values you list here MUST be the values that appear in the
code you then write.

(For `build`, present this as part of your plan; for `glowup`, present it with
the fixes you're about to apply. It does not require a separate user approval
unless the skill's own checkpoint calls for one — but it must be visible.)

---

## 8. Quick self-check before you send / write

Run this list every time — it's short on purpose:

- [ ] Did I do the §0 reasoning pre-flight (goal, refs, sections, icons, shadcn,
      risk)?
- [ ] Did I READ the `references/dna/<slug>.json` for every chosen slug, and show
      the §7 proof-of-extraction "Design DNA I'm using" block?
- [ ] Anti-slop (§6a): did I reproduce the reference's real `layout.hero_layout`
      (NOT the centered-hero-of-doom by default), land ≥1 signature move from
      `composition_techniques`, and introduce some asymmetry/tension?
- [ ] Do the hard floors (§6) all pass — `py-20`+ sections, `text-5xl`+ hero,
      ≥2 real motions from the DNA, 0 emoji, one radius + one shadow token?
- [ ] Zero emoji in the output AND in my reply? Icons used instead?
- [ ] Can I name the exact reference slug + the exact thing I pulled for each
      major section (styling, component shape, AND animation)?
- [ ] If interactive components were needed: did I use shadcn (React/Next) or its
      pattern (HTML), re-skinned to the palette?
- [ ] Human summary + options/question in my reply, in the user's language?
- [ ] If this skill has a HARD STOP, did I actually stop?

If any box is unchecked, fix it before sending.
