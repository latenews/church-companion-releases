# Shared ad feed

Static replacement for the retired `154.66.198.71:8001` ad server (FastAPI backend).
Served for free over HTTPS via `raw.githubusercontent.com` — this repo must stay
**public** or client apps can't fetch it without a token.

## Files

- `ads.json` — the full ad list. Every consuming app fetches this whole file and filters
  it client-side (there is no server to do `?app_id=...` filtering anymore).
- `images/` — the ad creative files referenced by `ads.json`'s `image` field (relative
  path from the repo root).

## `ads.json` schema

```json
{
  "id": 9,
  "target_app": "customer-companion-basic-android",
  "caption": "",
  "image": "images/some-file.png",
  "link_url": "https://example.com/",
  "active": true
}
```

- `target_app`: `null` means "show in every app". Otherwise must exactly match one of
  the `app_id` values apps send (see each app's `src/adFeed.ts`, e.g.
  `customer-companion-basic` / `customer-companion-basic-android` /
  `church-companion` / `church-companion-android`).
- `link_url`: `null` if the ad isn't clickable.
- `active`: set `false` to pull an ad without deleting its row/history.

## Publishing a new ad

**Driver Guardian Ad Studio (recommended)** — the operator Electron GUI was retrofitted
2026-09-02 to write here directly via the GitHub Contents API instead of the retired VPS.
Open it, Settings -> set Repository to `latenews/church-companion-releases`, Branch
`main`, and a GitHub personal access token with write access to this repo (classic PAT
with `repo` scope, or a fine-grained token scoped to Contents: Read and write on just
this repo). Every publish/edit/delete becomes one commit here, same as before conceptually
— just a different backend.

**Manual fallback** — for a one-off edit, or if Ad Studio isn't handy:
1. Add the image file under `images/`.
2. Append an entry to `ads.json` (bump `id` past whatever's already there).
3. `git add -A && git commit -m "..." && git push`

Either way, apps re-fetch on a 5-minute poll (see `POLL_MS` in `adFeed.ts`), so a new
ad shows up without needing a new app release.

## URLs consuming apps use

- Manifest: `https://raw.githubusercontent.com/latenews/church-companion-releases/main/ads/ads.json`
- Images: `https://raw.githubusercontent.com/latenews/church-companion-releases/main/ads/<image path from ads.json>`

Both are plain HTTPS, so this sidesteps the Android WebView "Mixed Content" block that
the old plain-HTTP VPS endpoint hit (Capacitor serves app pages at `https://localhost`,
which refuses to fetch insecure `http://` resources).
