# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Maintenance:** If this file exceeds 200 lines, refactor it using the `claude-md-refactor` skill.

## Multi-agent collaboration

**Multiple Claude instances may be working in this repo at the same time.** Stay out of each other's way.

### Before you start work
- Run `git status` and `git log --oneline -10`. If you see uncommitted changes, untracked files, or unfamiliar recent commits, **another agent may be active on a related area**. Identify which files belong to your task vs. theirs.
- If you've been asked to work on a feature and a directory or component for that feature already exists with changes you didn't make, **stop and ask the human** before proceeding. It may be the other agent's in-progress work.

### While working
- **Only modify files relevant to your task.** Don't reformat, refactor, or "clean up" files you didn't need to touch.
- **Never edit a file that already has uncommitted changes you didn't make.** If your task forces you to touch such a file, stop and ask the human how to coordinate. The other agent may be mid-edit and you'll create merge conflicts they can't recover from.
- **Don't run `git stash pop`, `git checkout --`, `git reset`, or similar destructive commands** on changes you didn't create. If you need to switch branches and the working tree is dirty with someone else's work, stash with `git stash push -u -m "wip-during-<your-task>"`, do your operation, and `git stash pop` immediately after — restoring exactly what was there.
- **Don't fix bugs in code you didn't write this session**, even if you spot one. Tell the human what you found and where, then move on. The other agent may already be addressing it, or the "bug" may be intentional WIP.

### When committing
- Run `git status` immediately before `git add`. **Only stage files you intentionally modified.**
- Never use `git add -A` or `git add .` if there are unstaged changes you didn't author. Stage by explicit path.
- Read your own diff with `git diff --staged` before committing. If anything in there isn't from your task, unstage it.
- If a `git pull` or branch switch reveals new commits you didn't make, **investigate before continuing**. Another agent may have shipped something that affects your work.

### Why all this
Git can't undo a commit that overwrote someone else's uncommitted edits — that work is gone. Most cross-agent damage happens at commit time, not edit time. Be paranoid at the staging step.

## What this is

**For / Not For Me** — a free, zero-hosting-cost web app for making "life-preference" line charts (how much some thing is *for you* across a lifetime), inspired by [Ty from the Internet's "Not Everything Is For You"](https://tyfromtheinternet.com/not-everything-is-for-you/). Users drag a line into shape and download a PNG. Deployed to Cloudflare Pages at `formecharts.jaygoldman.com`.

## Architecture — read this before editing

**The entire app is one file: `index.html`.** Inline `<style>` and `<script>`, no dependencies, no build step, no framework, no external network requests (so it works offline and stays truly free to host). Do not add a bundler, `package.json`, or npm dependencies unless the human explicitly asks — it would defeat the point.

Key pieces inside the `<script>` (all in one IIFE):

- **`state`** is the single source of truth: `{ title, emoji, maxAge, axis{yTop,yBottom,xLeft,xRight}, axisEmoji{yHigh,yLow,xLow,xHigh}, fonts{title,label}, colors{bg,grid,line,text}, xSteps, ySteps, points:[{y,corner}] }`. `y` is `0..1` (0 = bottom). There is one handle (`point`) on **every vertical age line**, so `points.length === xSteps + 1`; changing `xSteps` resamples `points` (this is enforced in `expand()` too). `ySteps` sets the horizontal divisions (percentage labels). `axisEmoji` are drawn at the axis endpoints; `fonts.title`/`fonts.label` are keys into the `FONTS` map.
- **Fonts** are export-safe stacks in the `FONTS` map. Two handwritten faces (Caveat, Patrick Hand) are **embedded as base64 woff2** in `<style id="fontface">` (latin subsets, fetched once from Google Fonts at authoring time — the app makes no runtime font requests). `svgString()` copies that `@font-face` CSS into the exported SVG so PNG/SVG downloads render with the chosen font. To add a font: add a system stack to `FONTS`, or embed another woff2 in `#fontface` and add its `FONTS` entry.
- **`render()`** is the one render path. Every edit mutates `state`, then calls `render()`, which rebuilds the SVG's `innerHTML`, updates the shareable URL (`history.replaceState`), and saves to `localStorage`. If you add a feature, route it through `state` → `render()` — don't patch the DOM out-of-band.
- **`linePath(P)`** builds the line's `d`. A segment is a straight `L` if either endpoint is a `corner`; otherwise a smooth Catmull-Rom cubic (`C`) with control-point Y clamped to the plot box so it can't overshoot past 0/1. This is what makes "click a handle → sharp corner" work.
- **Interaction** uses Pointer Events delegated on the `<svg>` (mouse + touch). Drag = change `y`; a press that doesn't move (< 3px) = click = toggle that point's `corner`. Pointer capture is on the `<svg>` root, **not** the handle — handles are destroyed on every re-render, so capturing them would break mid-drag.
- **Handles** carry class `export-hide`; `svgString()` strips those before PNG/SVG export so exports show only the chart.
- **Share links** encode `compact(state)` as base64url in the `#s=` hash; `expand()` reverses it. On load, URL hash wins over `localStorage`.

### Geometry
Fixed `viewBox="0 0 900 600"`; margins in the `M` object define the plot box. Use the helpers (`colX`, `valY`, `yToVal`, `clamp`) rather than recomputing coordinates.

## Verifying changes

No test suite — verify by hand in a browser (`open index.html`). Confirm: line starts flat at 50%, dragging redraws live, clicking a handle toggles a sharp corner, every control updates the chart, presets load, PNG/SVG/clipboard export work, and a copied share link reopens the exact chart in a fresh tab. The Claude-in-Chrome tools are useful for a visual pass.

## Deploy

Deployed as a **Cloudflare Worker using Static Assets** (not classic Pages). There is no Worker script — `wrangler.toml` has an `[assets]` block pointing at the repo root, and Cloudflare's build runs **`npx wrangler deploy`** to upload and serve the files. `.assetsignore` keeps repo meta files (README, LICENSE, wrangler.toml, etc.) from being served. Every push to `main` redeploys. **Don't switch the deploy command to `wrangler pages deploy`** — that targets a different resource type. See `README.md` for setup.

Custom domain `formecharts.jaygoldman.com` is attached to the Worker (**Settings → Domains & Routes**); the zone is on Cloudflare, so the DNS record + cert are automatic.

## Conventions

- **Credit Ty prominently.** The header, footer, and README link to the source post. Keep that visible in any redesign.
- Keep it dependency-free and self-contained. Embed assets inline; no CDN links, no external fonts.
- Match the existing terse, vanilla-JS style in `index.html` — no framework idioms.
