# getokui

A Claude Code plugin that gives Claude a **taste library** — a curated
collection of HTML design references (local, offline) — so AI-generated UIs
stop looking *template-y*.

getokui gives you **two flows**, both anchored to the taste library:

- **Create from scratch** — *brainstorming → build.* Claude reads the reference
  index, suggests the 5 best-matching candidates, you pick (you can pick more
  than one), then Claude adapts them into your own UI — in **HTML** (default),
  **React 19**, or **Next.js 15**.
- **Improve what you already have** — *review → glowup.* Point Claude at an
  existing UI file; it critiques the design **against the curated references**
  (spacing, color, typography, hierarchy), then glows it up — restyling your UI
  while keeping your content.

Different from a plain design-advice plugin: getokui has **real examples** to
reference + an agent that picks, adapts, reviews & improves — not just text that
says "make it look good".

---

## Architecture: one bundled repo

| Part | Contents | Size |
|---|---|---|
| Plugin | manifest + 5 skills (`setup`, `brainstorming`, `build`, `review`, `glowup`) | Small |
| `references/` (bundled) | Library: `index.json`, 213 `templates/*.html`, `thumbs/*.webp` | ~28MB |

The library **ships inside the plugin** under `references/`, so a single
`/plugin install` pulls everything — no separate clone, works offline
immediately. To get newer templates you just update the plugin (`/plugin
marketplace update`). Skills read the bundled files via
`${CLAUDE_PLUGIN_ROOT}/references/`.

---

## Install

Run in **order** — don't jump to `/plugin install` before
`/plugin marketplace add` finishes:

```
/plugin marketplace add hilmianugrah11/getokui
/plugin install getokui@getokui-marketplace
/reload-plugins
```

**Verify it actually installed** (not just the marketplace being added): open
`~/.claude/settings.json` and confirm there's
`"getokui@getokui-marketplace": true` under `enabledPlugins`. If it's not there,
`/plugin install` didn't run — adding the marketplace alone isn't enough.

> **Don't test by typing `/getokui` in the prompt.** The skills in this plugin
> are invoked **automatically** by Claude based on natural language, NOT via a
> manual slash command. So `/getokui` → "No commands match" is **normal**, not
> a sign of a failed install. The correct test: type a normal brief, e.g.
> *"getokui, build a SaaS landing page"*, and see whether the `brainstorming` skill triggers.

---

## How to use

The reference library is bundled with the plugin, so there's **nothing to set
up** — just start right after install. Two flows:

### A. Create from scratch — *brainstorming → build*

1. **Type a natural brief**, e.g.:
   > "getokui, build a SaaS landing page"
   > "getokui, build a dark fintech landing in react"
   > "getokui, build a restaurant landing in next"

2. **Follow the flow:**
   - The `brainstorming` skill presents **5 candidates** (name + description +
     thumbnail).
   - You **choose** — one, several (e.g. "layout #2, colors from #4"), or
     "search again" if none fit. **The agent stops here to wait for you** —
     this is the checkpoint.
   - The `build` skill mixes your picks → adapts → writes the file in your
     requested format.

### B. Improve what you have — *review → glowup*

1. **Point at a file**, e.g.:
   > "getokui, review index.html"
   > "getokui, cek design komponen Hero.tsx"

2. **Follow the flow:**
   - The `review` skill critiques the design **against the taste library** and
     presents ranked findings. **The agent stops here to wait for you** — this
     is the checkpoint.
   - You say **"glowup"** (or pick which findings) → the `glowup` skill applies
     the fixes to your file, keeping your content, and reports what changed.
   - You can also jump straight to *"getokui, glowup index.html"* to improve
     without a separate review first.

**Optional** — say *"setup getokui"* any time to confirm the library is present
and see the template count. To get newer templates, update the plugin (see
*Updating the plugin* below).

---

## Plugin structure

```
.claude-plugin/
  ├── plugin.json        ← plugin manifest
  └── marketplace.json   ← lets this repo be added as a marketplace
references/              ← bundled library (ships with the plugin)
  ├── index.json         ← metadata for every template (read by the agent)
  ├── templates/*.html   ← all 213 templates
  └── thumbs/*.webp      ← preview thumbnail per template
skills/
  ├── setup/SKILL.md          ← verify the bundled library + report template count
  ├── brainstorming/SKILL.md  ← read index → rank 5 candidates → checkpoint (hard stop)
  ├── build/SKILL.md          ← mix → adapt (style, not content) → convert format → output
  ├── review/SKILL.md         ← critique an existing UI vs the library → ranked findings (hard stop)
  └── glowup/SKILL.md         ← apply the fixes to your file (style, keep content) → report
```

**Two flows:** `brainstorming → build` (create new) and `review → glowup`
(improve existing). `setup` is a silent utility that just verifies the bundled
library.

---

## Core principles

- **The agent suggests, the user decides.** Reference selection always goes
  through a checkpoint — the agent stops and waits, never auto-builds. (Passes
  the agentic requirement + keeps demos safe.)
- **Take the style, not the content.** From a template it takes
  colors/spacing/structure/component shapes — NOT the original headlines/copy/
  brand assets. Sections the user hasn't filled in → clearly-marked
  placeholders.
- **Offline & curated.** The library is bundled with the plugin, quality
  guaranteed (manual curation). Demos don't depend on internet.
- **Always the latest versions.** React/Next output always uses the latest 2026
  stack (React 19, Next.js 15, Tailwind v4).

---

## Output formats

| Format | Stack | When to use |
|---|---|---|
| **HTML** (default) | HTML + Tailwind CDN | Zero-setup, just open the file. Easiest to demo. |
| **React** | React 19 · Vite 6 · TypeScript · Tailwind v4 | Need interactive components / React app integration. |
| **Next.js** | Next.js 15 (App Router) · TypeScript · Tailwind v4 | Need routing / SSR / production-ready. |

Mention the format in your brief (e.g. "...in react"); defaults to HTML if not
mentioned.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Typing `/getokui` → "No commands match" | **Normal** — skills are invoked via natural language, not a slash command | Test with a normal sentence, e.g. "getokui, build a SaaS landing page" |
| Skill doesn't trigger even with a normal sentence | Plugin not installed (only the marketplace was added) | Check `~/.claude/settings.json` → `enabledPlugins` must have `"getokui@getokui-marketplace": true` |
| "Library not found" / index.json missing | Broken/partial install — bundled `references/` didn't land | Reinstall: `/plugin install getokui@getokui-marketplace` then `/reload-plugins` |
| Clone fails | No connection / wrong repo | Check internet, confirm the references repo URL is correct, then retry |
| Thumbnail doesn't show | webp file missing / didn't render | Not a problem — candidates are still presented as text (name + description) |

---

## Updating the plugin

After editing this plugin's files & pushing to GitHub — **update the
marketplace first, then reload**:

```
/plugin marketplace update
/reload-plugins
```

If you bumped the version in `plugin.json` but the old cache is still in use,
uninstall then reinstall:

```
/plugin uninstall getokui
/plugin install getokui@getokui-marketplace
/reload-plugins
```

---

*Built for the Internal AI Agent Hackathon 2026. Pipeline pattern & checkpoint
discipline inspired by [`ajisss/design-agent`](https://github.com/ajisss/design-agent-plugin).*
