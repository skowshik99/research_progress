# Research Progress

A shareable, **script-free static website** summarizing *what is currently being
run* and visualizing experiment progress (dataset prep, trainings, evals,
SWE-bench trajectory generation). A landing page of project cards links to one
page per project, each holding dated entries with static visualizations only:
metrics tables, run-status tables, pre-rendered PNG plots, and embedded
self-contained example HTML (sandboxed `<iframe>`). **No JavaScript, no CDN.**

## Layout

```
index.html                     # landing: hero + project cards
style.css                      # the one stylesheet (dark theme)
.nojekyll                      # GitHub Pages serves raw HTML verbatim
projects/
  <slug>/
    index.html                 # header + dated entries (newest first)
    assets/                    # PNG plots + embedded example .html
```

## How it is maintained

This repo is edited **by hand** (no build step). The format is captured as a
Claude skill, `research_progress_site_v1`, in the separate
`continual-learning-bench` codebase (`.claude/skills/research_progress_site_v1/`).
To add a project / entry / plot / table / dataset example, prompt Claude Code
(with that repo open) e.g. *"add an entry to the research progress site for …"*
and it applies the skill's spec + templates to edit these HTML files. The skill
never lives in this repo; this repo stays 100% static.

Insertion markers keep edits unambiguous — do not remove them:
- landing `index.html`: `<!-- PROJECTS:START -->` … `<!-- PROJECTS:END -->`
- each project page: `<!-- ENTRIES:START -->` … `<!-- ENTRIES:END -->`

## One-time hosting on GitHub Pages

The `gh` CLI is not available here, so create the remote manually:

1. Create a **public** GitHub repo (e.g. `research_progress`) via the web UI.
2. Wire up the remote and push `main`:
   ```bash
   cd /home/skowshik/cl/research_progress
   git remote add origin https://github.com/<user>/research_progress.git
   git push -u origin main
   ```
3. In the repo: **Settings → Pages → Build and deployment**: Source =
   *Deploy from a branch*, Branch = `main`, folder = `/ (root)`. Save.
4. The site goes live at `https://<user>.github.io/research_progress/`.
   `.nojekyll` ensures the raw HTML and `projects/…` paths serve verbatim.

Only public / SWE-bench-derived content is ever committed here — never private
`/data/...` dumps, credentials, or multi-MB context files.
