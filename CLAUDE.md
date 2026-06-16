# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static **Kodi addon repository** (`delulu710/kodi-repo`) served via GitHub Pages and jsDelivr CDN. It contains no source code or build tooling — it's a hand-maintained distribution tree of addon metadata and pre-built zips that Kodi installs read directly over HTTP. The actual addon source lives outside this repo, in `/home/lho/cocoscrapers-fork/script.module.cocoscrapers/` (a fork of CocoScrapers renamed to "DeLuLuScraper Module").

## Structure

- `addons.xml` — concatenation of every addon's `addon.xml`, used by Kodi to discover available addons/versions. Must stay byte-identical to the individual `addon.xml` files it aggregates.
- `addons.xml.md5` — MD5 checksum of `addons.xml` (no trailing newline/filename, just the hex digest). Kodi compares this before re-downloading `addons.xml`.
- `repository.delulu710/` — the repository addon itself (the thing users install first to point Kodi at this repo).
  - `addon.xml` — must contain a `<dir>` wrapper around `<info>`/`<checksum>`/`<datadir>` for Kodi 19+ (Matrix) compatibility; without it Kodi silently ignores those fields.
  - `repository.delulu710-1.0.0.zip` — zip containing just `repository.delulu710/addon.xml`, named `<id>-<version>.zip`.
- `script.module.cocoscrapers/` — the DeLuLuScraper Module addon: `addon.xml`, `icon.png`, `fanart.png`, and `script.module.cocoscrapers-<version>.zip` (the full built addon, source synced from the `cocoscrapers-fork` repo).
- `index.html` per directory — directory listing pages so Kodi's HTTP file browser can list zip contents (no native directory index on GitHub Pages).
- `.nojekyll` — disables Jekyll processing on GitHub Pages so files starting with `.`/underscored paths aren't filtered.

## Updating an addon (release workflow)

There is no build script — releases are done by hand:

1. Bump the `version` attribute in the addon's `addon.xml` (must match the new zip filename).
2. Build the new zip as `<addon-id>/<addon-id>-<version>.zip`, containing the addon's full folder (matching the addon's internal layout), and place it in that addon's directory.
3. Regenerate `addons.xml` by concatenating all current `addon.xml` files under `<addons>...</addons>`.
4. Regenerate `addons.xml.md5`: `md5sum addons.xml | awk '{print $1}' > addons.xml.md5` (digest only, no filename).
5. Commit and push to `master` — GitHub Pages and jsDelivr serve directly from the repo (jsDelivr CDN has its own cache lag on top of GitHub Pages).

## Addon URLs

- Repo root (GitHub Pages): `https://delulu710.github.io/kodi-repo/`
- Repository zip (what users add to Kodi as a file source then install from): `https://delulu710.github.io/kodi-repo/repository.delulu710/repository.delulu710-1.0.0.zip`
- jsDelivr CDN mirror (used inside `repository.delulu710/addon.xml` for `info`/`checksum`/`datadir`, for better compatibility on devices like NVIDIA Shield): `https://cdn.jsdelivr.net/gh/delulu710/kodi-repo@master/`

## Gotchas learned from past debugging

- Kodi 19+ requires the `<dir>` wrapper in the repository addon's `<extension point="xbmc.addon.repository">` block — omitting it makes Kodi unable to find the repo's `addons.xml`/checksum/datadir even though the URLs are otherwise correct.
- `raw.githubusercontent.com` URLs caused compatibility issues on some devices (e.g. NVIDIA Shield); jsDelivr's CDN URLs are more broadly compatible.
- `addons.xml` must always match the sum of the individual `addon.xml` files exactly, and `addons.xml.md5` must be regenerated every time `addons.xml` changes — Kodi uses the checksum to decide whether to re-fetch.
