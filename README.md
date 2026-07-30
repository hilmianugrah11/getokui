# getokui

A Claude Code plugin that gives Claude a **taste library** — a curated
collection of HTML design references (local, offline) — so AI-generated UIs
stop looking *template-y*.

When you ask it to build a UI, Claude **doesn't make things up from scratch**:
it reads a reference index, suggests the 5 best-matching candidates, you pick
(you can pick more than one), then Claude adapts them into your own UI — in
**HTML** (default), **React 19**, or **Next.js 15**.

Different from a plain design-advice plugin: getokui has **real examples** to
reference + an agent that picks & adapts, not just text that says "make it look
good".

---

## Architecture: two repos

| Repo | Contents | Size |
|---|---|---|
| **getokui** (this) | Plugin: manifest + 3 skills (`setup`, `pick`, `build`) | Small |
| **getokui-references** | Library: `index.json`, `templates/*.html`, `thumbs/*.webp` | ~1–2MB |

The plugin is installed once. The library is cloned to `~/.getokui/references/`
and can be updated independently (`git pull`) without reinstalling the plugin.

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
> *"getokui, build a login page"*, and see whether the `pick` skill triggers.

---

## How to use

1. **Once, upfront** — prepare the reference library. Type a normal sentence:
   > "setup getokui"

   The `setup` skill will `git clone` the library to `~/.getokui/references/`.
   Needs internet just this once; after that it works offline.

2. **Start building UI** — type a natural brief, e.g.:
   > "getokui, build a login page"
   > "getokui, build a dark SaaS landing page in react"
   > "getokui, build a dashboard in next"

3. **Follow the flow:**
   - The `pick` skill presents **5 candidates** (name + description +
     thumbnail).
   - You **choose** — one, several (e.g. "layout #2, colors from #4"), or
     "search again" if none fit. **The agent stops here to wait for you** —
     this is the checkpoint.
   - The `build` skill mixes your picks → adapts → writes the file in your
     requested format.

4. **Update the library** anytime:
   > "update getokui references"

---

## Plugin structure

```
.claude-plugin/
  ├── plugin.json        ← plugin manifest
  └── marketplace.json   ← lets this repo be added as a marketplace
skills/
  ├── setup/SKILL.md      ← clone/update the reference library (~/.getokui/references)
  ├── pick/SKILL.md       ← read index → rank 5 candidates → checkpoint (hard stop)
  └── build/SKILL.md      ← mix → adapt (style, not content) → convert format → output
```

---

## Core principles

- **The agent suggests, the user decides.** Reference selection always goes
  through a checkpoint — the agent stops and waits, never auto-builds. (Passes
  the agentic requirement + keeps demos safe.)
- **Take the style, not the content.** From a template it takes
  colors/spacing/structure/component shapes — NOT the original headlines/copy/
  brand assets. Sections the user hasn't filled in → clearly-marked
  placeholders.
- **Offline & curated.** The library is cloned once, quality guaranteed (manual
  curation). Demos don't depend on internet.
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
| Typing `/getokui` → "No commands match" | **Normal** — skills are invoked via natural language, not a slash command | Test with a normal sentence, e.g. "getokui, build a login page" |
| Skill doesn't trigger even with a normal sentence | Plugin not installed (only the marketplace was added) | Check `~/.claude/settings.json` → `enabledPlugins` must have `"getokui@getokui-marketplace": true` |
| "Library not found" / index.json missing | Haven't run `setup` | Type "setup getokui" first to clone the library |
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
