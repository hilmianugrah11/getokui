# getokui — Build Spec (Claude Code Plugin)

**Date:** 2026-07-30
**Status:** Design — ready for review
**Event:** Internal AI Agent Hackathon (deadline July 31, 2026)
**Visual spec:** open `docs/superpowers/specs/2026-07-30-getokui-visual-spec.html` in a browser.

---

## 1. Summary

**getokui** is a Claude Code plugin that gives Claude a *"taste library"*: a
curated collection of HTML design references. It offers **two flows**, both
anchored to that library:

- **Create from scratch (`brainstorming → build`)** — the agent
  **doesn't invent from scratch**: it reads a local reference index, suggests
  the 5 best-matching candidates, the user picks (can pick more than one), then
  the agent adapts them into the user's own UI in the requested format
  (HTML / React / Next.js).
- **Improve what exists (`review → glowup`)** — the user points at an existing
  UI file; the agent critiques its design **against the curated references**,
  then glows it up (restyle, keep the user's content).

Problem it solves: AI-generated UIs often look *template-y* (plain, generic)
because the AI guesses from scratch. Tools like Lovable/v0 are good because they
have a reference collection to draw on. getokui gives Claude that collection —
local and curated.

---

## 2. Architecture: One Bundled Repo

The reference library ships **inside the plugin** under `references/`. A single
`/plugin install` pulls the manifest, the five skills, and the whole library
together — one repo, one install step, works offline immediately. The library is
only ~2MB, so bundling keeps install trivially fast while removing the separate
clone that could fail during a live demo.

```
getokui/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # so it can be installed via a GitHub marketplace
├── references/              # BUNDLED library — ships with the plugin
│   ├── index.json           # metadata for all templates (read by the agent)
│   ├── templates/
│   │   └── <slug>.html      # 20 curated templates
│   └── thumbs/
│       └── <slug>.webp      # self-screenshotted, WebP, compressed
└── skills/
    ├── setup/SKILL.md          # verify the bundled library + report count
    ├── brainstorming/SKILL.md  # read index → rank 5 candidates → checkpoint
    ├── build/SKILL.md          # mix → adapt → convert format → output file
    ├── review/SKILL.md         # critique an existing UI vs the library → checkpoint
    └── glowup/SKILL.md         # apply fixes to the user's file (style, keep content)
```

Skills read the bundled files via the `${CLAUDE_PLUGIN_ROOT}/references/` path
(`${CLAUDE_PLUGIN_ROOT}` is the plugin's install directory — an official Claude
Code plugin variable that expands in skill instructions).

**Why bundled (not a second repo):** one public repo, one install step, no
network dependency at demo time. Newer templates ship by updating the plugin
(`/plugin marketplace update`) — the user doesn't manage a separate library
folder.

> **History:** an earlier design used two repos (`getokui` +
> `getokui-references`) with a `setup` skill that `git clone`d the library to
> `~/.getokui/references/`. That was collapsed into this single bundled repo to
> simplify install and de-risk the demo. The `setup` skill was repurposed from
> "clone" to "verify the bundled library".

---

## 3. Skill Pipeline (adapted from `design-agent`)

Instead of one big `SKILL.md`, getokui uses a **five-skill pipeline** across
**two flows** — each skill with one clear job, with a **checkpoint = hard stop**
before any action. This pattern is adapted from the `ajisss/design-agent` plugin
(`init → inspo → select → spec → build`), reshaped for the hackathon scope.

**Flow A — create from scratch (`brainstorming → build`):**

| Skill | Job | Checkpoint |
|---|---|---|
| **setup** | Verify the bundled library at `${CLAUDE_PLUGIN_ROOT}/references/` + report the template count. "update references" → update the plugin. | — (silent utility) |
| **brainstorming** | Parse request → filter `index.json` → present **5 ranked candidates** (name+description+thumbnail). | **HARD STOP** — wait for the user to choose. Won't continue to build on its own. |
| **build** | Mix picks → adapt (style, not content) → convert format → write file. | Confirm format if ambiguous. |

**Flow B — improve what exists (`review → glowup`):**

| Skill | Job | Checkpoint |
|---|---|---|
| **review** | Read the file the user points at → critique its design **against the library** (spacing/color/type/hierarchy/polish/consistency) → present **ranked findings**. Reads only. | **HARD STOP** — wait for the user to approve fixes (all / some / none). |
| **glowup** | Apply the approved fixes to the user's file — pull **style** from 1–3 refs, **keep the user's content** → edit in place (or a `.glowup.` copy) → report changes. | Can run directly ("glow this up") with a quick internal review first. |

**Checkpoint principle (from design-agent):** a skill with a checkpoint MUST
fully stop and wait for the user's explicit answer in the next message. Don't
continue on your own assumption — even when the answer seems "obvious". This is
what keeps the user in control (safe demos) + satisfies the agentic requirement
(there's a decision, not an auto-run).

---

## 4. Distribution & Installation

**Via a GitHub Marketplace.** These instructions follow the pattern proven in
`design-agent` — run in **order**, don't skip:

```
/plugin marketplace add hilmianugrah11/getokui
/plugin install getokui@getokui-marketplace
/reload-plugins
```

**Verify it actually installed** (not just the marketplace being added): open
`~/.claude/settings.json` and confirm `"getokui@getokui-marketplace": true` is
under `enabledPlugins`. If it's not there → `/plugin install` didn't run.

> **Don't test by typing `/getokui` in the prompt.** Skills are invoked
> automatically by Claude from natural language (e.g. *"getokui, build a SaaS
> landing page"*), NOT a manual slash command. `/getokui` → "No commands match" is
> **normal**, not a failed install.

The reference library is **bundled inside the plugin** (`references/`), so
`/plugin install` already puts all templates + thumbnails on disk — there's no
separate clone step. The `setup` skill just **verifies** the bundled library is
present and reports the template count:

```
${CLAUDE_PLUGIN_ROOT}/references/index.json   ← already there after install
```

- Install once → all templates + thumbnails are local (bundled).
- Safe demo: no internet needed at all — nothing is fetched at runtime.
- Update: newer templates ship by updating the plugin
  (*"update getokui references"* → `/plugin marketplace update` +
  `/reload-plugins`).

**Why bundle (not fetch per-request, not a second repo):** lighter at use time
(reads local files, no network latency), one public repo, one install step, and
demos don't depend on connectivity. Total library size is small (~2MB, see §7).
This is also the core difference from `design-agent`, which searches for live
references from the web on every request — getokui is offline & curated.

---

## 5. index.json — Reference Metadata

Detail level: **rich + thumbnail**. Each entry has enough metadata for the agent
to *match* without having to read the HTML first.

```json
{
  "version": 1,
  "templates": [
    {
      "slug": "aura-ai-landing",
      "name": "Aura AI Landing",
      "category": "landing",
      "tags": ["saas", "ai", "dark", "gradient", "hero"],
      "description": "SaaS AI landing page, dark theme, big hero + gradient, card-based features.",
      "colors": ["#0B0B12", "#7C5CFF", "#E9E9F0"],
      "sections": ["hero", "features", "pricing", "cta", "footer"],
      "fonts": ["Inter", "sans-serif"],
      "thumbnail": "thumbs/aura-ai-landing.webp"
    }
  ]
}
```

Fields:
- `slug` — unique id, same as the `.html` & `.webp` filename.
- `category` — main bucket for fast filtering: `landing`, `fintech`,
  `real-estate`, `portfolio`, `ecommerce`, `restaurant`, `wellness`, `gaming`,
  `agency`, `architecture`.
- `tags` — keywords for finer matching.
- `description` — one sentence, so the agent understands the "feel" without
  reading the HTML.
- `colors` — dominant palette (for the agent to mix style).
- `sections` — list of sections present (to match user needs).
- `thumbnail` — relative path to the local WebP file.

---

## 6. Agent Workflow

```
1. User asks           "getokui, build a SaaS landing page"
        │
2. Parse request       detect: category=landing, format=HTML (default)
        │
3. Filter index.json   pull candidates by category + tags
        │
4. Present 5 candidates ranked, show name + description + thumbnail
   (multi-select)      user may pick >1, may reject all & ask for others
   ══ HARD STOP ══     agent STOPS, waits for the user
        │
5. Mix into 1 result   first pick = base layout,
                       other picks = style sources (colors/fonts/components)
        │
6. Adapt               take STYLE + STRUCTURE, not the template's text content
        │
7. Convert format      HTML / React 19 / Next.js 15 as requested
        │
8. Output file         write the finished file + tell the user the path
```

**Selection details (agentic):**
- The agent finds the **5 most relevant** candidates from the index, *ranked*.
- The user can **pick more than one**.
- The user can **pick outside** the agent's suggestions (reject all → agent
  searches again / shows another category).
- If multi-pick: **mix into 1 result** — first pick becomes the base layout,
  other picks become style sources (colors, fonts, specific components).

**Honest about match quality (from design-agent):** if no candidate is a real
fit, the agent says so plainly ("nothing's a perfect fit for X, these are the
closest") and offers to search again — it doesn't force a weak candidate as if
it fit.

**Flow B — improve what exists (`review → glowup`):**

```
1. User points at a file   "getokui, review index.html"
        │
2. Read the target         parse markup/JSX + styling (Tailwind/CSS/tokens)
        │
3. Pick 1-3 refs           choose comparison references from index.json
        │
4. Evaluate                spacing · color · typography · hierarchy ·
                           component polish · consistency — vs the refs
        │
5. Present ranked findings each: element/line — what's weak → the fix
   ══ HARD STOP ══         (ref: <slug> does this well)
        │                  agent STOPS, waits for the user's go
        │
6. glowup (on approval)    apply the approved fixes — pull STYLE from the refs,
                           KEEP the user's content
        │
7. Edit in place           (or a `.glowup.` copy) → report what changed
```

Review reads only; it never edits. glowup can also run directly ("glow this up")
with a quick internal review first. Neither writes into the bundled
`${CLAUDE_PLUGIN_ROOT}/references/` — output always goes to the user's file.

---

## 7. Curation & Thumbnail Pipeline

References are prepared **manually** (quality guaranteed, stable demos). Initial
source: 213 templates scraped in `template/aura` — pick the **15-25 best & most
varied** (landing pages across verticals: SaaS/AI, fintech, real-estate,
portfolio, e-commerce, restaurant, wellness, gaming, agency, architecture).

**Thumbnails are made by us (NO supabase / external URLs):**

1. Open each curated HTML template with **Playwright headless** (local file).
   - Playwright is already a dependency in `template/aura/package.json`
     (^1.61.1).
2. Screenshot the desktop viewport **1280×800**, waiting for render to settle.
3. Convert to **WebP** + compress with **Pillow** (quality ~80, resize to
   ~640px wide).
4. Save to `thumbs/<slug>.webp`.

Result: ~20-50KB per thumbnail. 25 thumbnails ≈ ~1MB. The library repo stays
light.

**Why screenshot ourselves:** all assets are our own production, living in the
repo, bundled with the plugin along with everything else. No dependency on an
external URL that could die/change.

---

## 8. Output Formats

The user can choose the output format (default: HTML). Always uses the **latest
2026 versions**.

| Format | Stack | Why |
|---|---|---|
| **HTML** (default) | HTML + Tailwind CDN | Zero-setup, just open the file — easiest to preview & demo. |
| **React** | React 19 + Vite 6 + TypeScript + Tailwind v4 | For interactive components / React app integration. |
| **Next.js** | Next.js 15 (App Router, Server Components) + TypeScript + Tailwind v4 | For routing/SSR/production-ready. |

The agent detects the format from the request (e.g. "build a landing **in
react**"), falling back to HTML if unspecified.

---

## 9. Adaptation Rule: Style vs Content (from design-agent)

An important rule so adaptation isn't just plagiarism:

**What MAY be taken from the reference template:**
- Visual style: colors, spacing, radius, shadow, typography, motion.
- Component structure/shape: layout grid, card shape, button shape, section
  types (hero, features, pricing).

**What may NOT:**
- The template's original text/copy (headlines, product descriptions, feature
  names) as if it were the user's content. If the user hasn't provided content
  for a section, use a clearly-marked placeholder (e.g. `[Product headline
  here]`) or ask first.
- Reproducing third-party brand assets exactly (logos, photos, original
  illustrations). Take the **structural pattern**, not the visual asset.

Since we curate the templates ourselves (not live competitor products),
copyright risk is low — but the style-vs-content rule still holds so the result
genuinely "belongs to the user", not a paste-up.

---

## 10. Error Handling

| Condition | Agent behavior |
|---|---|
| Bundled library missing (broken/partial install) | Tell the user plainly + suggest reinstalling the plugin (`/plugin install` → `/reload-plugins`). Don't stay silent. |
| Category not found in index | Offer the nearest category / related templates, don't force it. |
| Thumbnail missing/corrupt | Keep going — present candidates using text (name + description) only. |
| User rejects all candidates | Search again with looser filters / show another category. |
| Weak match (nothing fits) | Admit it plainly, offer the closest option + search again. |

---

## 11. Testing (pre-demo)

Before the presentation, test **3 end-to-end scenarios**:

1. `"getokui, build a SaaS landing page"` → **HTML** output done & openable.
2. `"getokui, build a fintech landing in react"` → valid **React 19** output.
3. `"getokui, build a restaurant landing in next"` → valid **Next.js 15** output.

Passes if: all three produce correctly-formatted files, look clearly better than
plain AI output, and the brainstorming flow works (5 candidates appear, hard
stop, multi-pick, mix). Flow B check: `"getokui, review index.html"` returns
ranked findings anchored to named refs and stops; `"glowup"` then restyles the
file while keeping its content.

---

## 12. Success Criteria

Given the command `"getokui, build a SaaS landing page"`, the agent can:

1. Read `index.json` from the local library, filter candidates in the `landing`
   category.
2. Present **5 ranked candidates** (name + description + thumbnail), **stop** and
   wait for the user; the user can multi-pick / reject all.
3. Mix the picks → adapt (style, not content) → write the finished **HTML file**
   to disk.
4. Detect the format if the user asks for React/Next.js, output the correct
   latest-version stack.
5. On `"getokui, review <file>"`, critique the file **against named library
   refs**, present ranked findings, and **stop** — then, on approval, `glowup`
   applies the fixes in place while keeping the user's content.

**Passing criteria:** all four steps run without crashing, the resulting UI is
clearly more *polished* than reference-less AI output, the checkpoint flow works
(agent stops at candidates, doesn't auto-build), and the whole process can be
demoed offline (library bundled with the plugin).

---

## 13. What's Adapted from `ajisss/design-agent`

For transparency (we learned from a plugin that already works):

| design-agent pattern | Used in getokui? | Note |
|---|---|---|
| Multi-skill pipeline (`init→inspo→select→spec→build`) | ✅ Yes (reshaped → two flows: `setup→brainstorming→build` + `review→glowup`) | 5 skills across create + improve. |
| Checkpoint = hard stop | ✅ Yes | Between `brainstorming` and `build`, and between `review` and `glowup`. |
| Separate style vs content | ✅ Yes | §9. |
| Battle-tested marketplace install instructions | ✅ Yes | §4, including the "don't test via slash command" note. |
| Honest about uncertainty (confidence) | ✅ Partly | Applied as "honest about match quality" (§6), without a full confidence-marker system. |
| Copyright note (structural pattern, not assets) | ✅ Yes | §9. |
| File-based registry (`.design/registry/*.json`) | ➖ No (optional next) | getokui doesn't need per-project state; its library is global. |
| Hooks (PostToolUse validate-tokens.py) | ➖ No | Overkill for one day; no crash risk at demo. |
| QA token-compare + visual diff (Playwright) | ➖ No | getokui uses Playwright only to generate thumbnails, not runtime QA. |

**Core difference:** design-agent **searches for live references from the web**
(Dribbble/Mobbin/Land-book) each request; getokui uses a **curated local
library** bundled with the plugin — offline, demo-safe, quality guaranteed.

---

## 14. Open Questions (out of initial build scope)

- Dashboard preview (gallery on the left + iframe on the right) — **deferred**,
  focus on the skill first for a safe demo (no server that can crash).
- Final number of curated templates (15 or 25) — decide when filling the
  library.
- How far "adaptation" goes (swap text+colors only, or allow layout changes) —
  default: layout changes allowed, agent uses judgment.
- Need a lightweight registry/journal to track used templates? — optional, next
  iteration.
