# Forster

A digital [Forster list](https://forum.beeminder.com/t/the-best-digital-implementation-of-forster-lists/12807) — compare one task at a time against a "benchmark" instead of scanning a long list. Static, single-file, no build step, no server.

## What this repo is (and isn't)

This repo contains **only app code** — `index.html` and the Pages deploy workflow. It has **no data in it, no credentials, and no reference to any specific person's task list**. Nothing here identifies whose data it's used with.

The published app is a "bring your own repo" tool. It stores a GitHub repo (owner/repo/branch/paths) and a personal access token, entered by whoever operates it, only in that browser's `localStorage`. From then on it reads/writes that repo's `tasks.json` (and optionally a vehicle-maintenance file) directly from the browser via GitHub's REST API — this repo and its Pages hosting are never part of that data path.

The landing screen deliberately gives no indication of any of this: no title, no labels, no instructions. It's a blank set of inputs in a fixed order known only to whoever set it up; nothing about the app, what it does, or what the fields mean is discoverable from the rendered page. (Reading the source still tells you what the code does, same as any static site — the point isn't code secrecy, it's giving a casual visitor nothing to look at.)

This repo being public is safe specifically because it never contains anyone's data — only the generic client-side app. Whoever's task data it points at is controlled entirely by what the operator types in, on their own device.

## Deploy

Push to `main` — `.github/workflows/deploy.yml` builds and publishes `index.html` to GitHub Pages.
