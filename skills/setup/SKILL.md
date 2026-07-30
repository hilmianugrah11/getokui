---
name: setup
description: Prepare or update the getokui reference library locally. Clone the getokui-references repo to ~/.getokui/references (once, upfront), or git pull when the user asks to "update getokui references". MUST run before the pick skill if the library folder doesn't exist yet. Triggers: "setup getokui", "install getokui references", "update getokui references", or automatically invoked by the pick skill when it detects the library isn't cloned yet.
---

# getokui setup — Prepare the Reference Library

This skill has exactly one job: make sure the getokui reference library exists
locally and is up to date. It does **not** pick templates and does **not**
generate UI — those are the jobs of `pick` and `build`.

Local library location (standard): `~/.getokui/references/`
Source: `https://github.com/hilmianugrah11/getokui-references`

## Steps

### 1. Check whether the library already exists
Check whether `~/.getokui/references/index.json` exists.

- **Already exists** → the library is installed. If the user only said "setup"
  (not "update"), tell them the library is ready plus the template count, then
  STOP. Don't re-clone or overwrite.
- **Doesn't exist** → continue to Step 2 (clone).

Use cross-platform commands (the agent may run on Windows/Mac/Linux). Example
check in a POSIX shell:
```bash
test -f "$HOME/.getokui/references/index.json" && echo EXISTS || echo MISSING
```
On Windows PowerShell:
```powershell
Test-Path "$env:USERPROFILE\.getokui\references\index.json"
```

### 2. Clone the library (if missing)
```bash
git clone --depth 1 https://github.com/hilmianugrah11/getokui-references "$HOME/.getokui/references"
```
Windows PowerShell:
```powershell
git clone --depth 1 https://github.com/hilmianugrah11/getokui-references "$env:USERPROFILE\.getokui\references"
```

If the clone **fails** (no connection / wrong repo / git not installed):
- Don't stay silent. Tell the user what the error is (network / repo not found
  / git missing).
- Offer options: check the connection and retry, or provide a local path
  manually if the user already has the folder themselves.
- Don't proceed to `pick` before the library actually exists.

### 3. Verify the clone
After cloning, make sure at minimum these exist:
- `~/.getokui/references/index.json`
- `~/.getokui/references/templates/` (folder of `*.html`)
- `~/.getokui/references/thumbs/` (folder of `*.webp`)

Read `index.json`, count the entries in `templates[]`, and report to the user:
> "getokui library ready — <N> templates cloned to ~/.getokui/references.
> Now just say something like 'getokui, build a login page'."

### 4. Update (when the user asks to "update getokui references")
```bash
git -C "$HOME/.getokui/references" pull --ff-only
```
Windows PowerShell:
```powershell
git -C "$env:USERPROFILE\.getokui\references" pull --ff-only
```
Report the changes (how many new templates, if visible from the output). If the
folder doesn't exist at all, treat it like Step 2 (clone first).

## What this skill must NOT do
- Pick / rank templates — that's `pick`'s job.
- Generate or write UI files — that's `build`'s job.
- Overwrite an existing library without reason (unless the user explicitly asks
  to re-clone/reset).
- Auto-continue to `pick` — confirm the library is ready first, and let the
  user start the next brief.
