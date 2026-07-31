---
name: setup
description: Verify the getokui reference library is present. The library is BUNDLED inside the plugin (ships with `/plugin install`), so there is no separate clone step — this skill just confirms the bundled references exist and reports the template count. Triggers: "setup getokui", "is getokui ready", "check getokui". For "update getokui references", explain that updating the library = updating the plugin (`/plugin marketplace update` then `/reload-plugins`).
---

# getokui setup — Verify the Bundled Library

This skill has exactly one job: confirm the getokui reference library is
available. Unlike the old two-repo design, the library now **ships inside the
plugin** — it's already on disk the moment the plugin is installed. There is
**no clone and no internet needed**. This skill does **not** pick templates and
does **not** generate UI — those are the jobs of `brainstorming` and `build`.

Bundled library location: `${CLAUDE_PLUGIN_ROOT}/references/`

Communication: follow `${CLAUDE_PLUGIN_ROOT}/skills/shared/DOCTRINE.md` §4 —
report in plain human language, no emoji, end with a clear next step.

## Steps

### 1. Verify the bundled library exists
Check that these are present under `${CLAUDE_PLUGIN_ROOT}/references/`:
- `index.json`
- `templates/` (folder of `*.html`)
- `thumbs/` (folder of `*.webp`)

POSIX shell:
```bash
test -f "${CLAUDE_PLUGIN_ROOT}/references/index.json" && echo EXISTS || echo MISSING
```
Windows PowerShell:
```powershell
Test-Path "$env:CLAUDE_PLUGIN_ROOT\references\index.json"
```

### 2. Report
- **Present** → read `index.json`, count entries in `templates[]`, and tell the
  user:
  > "getokui is ready — <N> reference templates bundled with the plugin. Just
  > say something like 'getokui, build a SaaS landing page'."
  Then STOP — let the user start their next brief.
- **Missing** (unexpected — the bundled files should always be there) → this
  means a broken/partial install. Tell the user plainly and suggest reinstalling
  the plugin:
  ```
  /plugin install getokui@getokui-marketplace
  /reload-plugins
  ```
  Don't proceed to `brainstorming` until the library is present.

### 3. "Update getokui references"
There is no separate library repo to `git pull` anymore — the library is part of
the plugin. To get newer templates, the user updates the **plugin**:
```
/plugin marketplace update
/reload-plugins
```
If the version was bumped and the old cache is still active, reinstall:
```
/plugin install getokui@getokui-marketplace
/reload-plugins
```
Explain this to the user rather than trying to clone or pull anything.

## What this skill must NOT do
- Clone or `git pull` anything — the library is bundled, not fetched.
- Pick / rank templates — that's `brainstorming`'s job.
- Generate or write UI files — that's `build`'s job.
- Auto-continue to `brainstorming` — confirm the library is ready first, and let the
  user start the next brief.
