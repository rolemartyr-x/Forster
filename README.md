# Forster

A digital [Forster list](https://forum.beeminder.com/t/the-best-digital-implementation-of-forster-lists/12807) — compare one task at a time against a "benchmark" instead of scanning a long list. Static, single-file, no build step, no server.

## Mechanics

Ordering is Bayesian Thompson sampling: each task keeps a `{yes, no, cant}` counter, and every draw samples `Beta(yes+1, no+1)` per eligible candidate and offers the highest sample. An unjudged task samples uniformly, so new work is never buried under an established favorite; `cant` is recorded but excluded from the Beta parameters ("not now," not "not ever"). This replaces the earlier fixed 4-hour No-cooldown and 14-day due-window cutoff — a low-scoring or far-off task stays eligible, just unlikely to be drawn.

There is no "Contexts" feature (Home/Laptop/Errand/Weekend/Car tags) anymore. Filtering is by type/priority/effort chips, hide-blocked/hide-waiting/started-only toggles, and free-text search.

Other things ported in: a comments log per task (oldest-first, `Ctrl+Enter` to submit), defer/sleep (presets, cap 365 days, rejects today-or-past), a dispose row on the candidate card (Already done / Cancel, archives without ever entering the chain), Undo on every close, and `Y`/`N`/`C`/`D` keybindings (archiving stays mouse-only). The chain itself now persists in the vault, not just `localStorage` — see "Data model" below.

All of this was ported from the mechanics of a separate, already-working local Python build used for reviewing work tasks — adapted to this repo's no-server, GitHub-Contents-API-backed architecture (chosen originally so the app works from both phone and PC; see "why it ended up this way" in the vault's own `02 Projects/forster-chain/README.md` for that history).

## Data model

`tasks.json` is untouched beyond what this app already wrote before (`completedAt` on Done/Cancel) — it's a file synced by another app on the owning vault, and this app doesn't know whether that sync tolerates unrecognized fields. Everything new (comments, `start`, `blocked_by`, `waiting_on`, `effort`) instead lives in three sidecar files, alongside `tasksPath`:

- `forster-scores.json` — `{taskId: {yes, no, cant, last_seen}}`
- `forster-chain.json` — `{date, chain: [ids], aside: {id: reason}}`
- `forster-meta.json` — `{taskId: {comments, start, blocked_by, waiting_on, effort}}`

Paths are configurable in Settings (default: same directory as `tasksPath`), and each is created on first write if it doesn't exist yet.

There's no server-side file lock in this architecture — writes are `PUT`s straight to the GitHub Contents API. On a stale-SHA conflict (409/422, e.g. phone and PC writing near-simultaneously) the app re-fetches the sidecar, merges onto it (comments arrays de-dupe by exact match, counters take the higher value, chain/meta docs take the more recent write), and retries once.

## What this repo is (and isn't)

This repo contains **only app code** — `index.html` and the Pages deploy workflow. It has **no data in it, no credentials, and no reference to any specific person's task list**. Nothing here identifies whose data it's used with.

The published app is a "bring your own repo" tool. It stores a GitHub repo (owner/repo/branch/paths) and a personal access token, entered by whoever operates it, only in that browser's `localStorage`. From then on it reads/writes that repo's `tasks.json` (and optionally a vehicle-maintenance file) directly from the browser via GitHub's REST API — this repo and its Pages hosting are never part of that data path.

The landing screen deliberately gives no indication of any of this: no title, no labels, no instructions. It's a blank set of inputs in a fixed order known only to whoever set it up; nothing about the app, what it does, or what the fields mean is discoverable from the rendered page. (Reading the source still tells you what the code does, same as any static site — the point isn't code secrecy, it's giving a casual visitor nothing to look at.)

This repo being public is safe specifically because it never contains anyone's data — only the generic client-side app. Whoever's task data it points at is controlled entirely by what the operator types in, on their own device.

## Deploy

Push to `main` — `.github/workflows/deploy.yml` builds and publishes `index.html` to GitHub Pages.
