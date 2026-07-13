# tunetap-site

GitHub Pages host for the **TuneTap** app's deep-link association files and a
web fallback for tag URLs. Served at **https://tunetap.to**.

Consumed as a git submodule (`site/`) by the main app repo (`musicbox`).

## What's here

| File | Purpose |
|------|---------|
| `.well-known/apple-app-site-association` | iOS Universal Links (AASA). Lets the app claim `https://tunetap.to/p/*`. |
| `.well-known/assetlinks.json` | Android App Links (Digital Asset Links). Verifies the app for the same host. |
| `.nojekyll` | Disables Jekyll so the dot-prefixed `.well-known/` directory is served (Jekyll ignores dot-dirs). **Do not delete.** |
| `CNAME` | Custom domain (`tunetap.to`) for GitHub Pages. |
| `index.html` | Landing page. |
| `404.html` | Fallback for `/p/{id}` when the app isn't installed — parses the playlist id and offers an Open-in-Spotify link. GitHub Pages serves this for any unmatched path. |

Tag URL shape (from the app's `TagLink`): `https://tunetap.to/p/{playlistId}?t={title}`
— so both association files scope to path **`/p/*`**.

## ⚠️ Placeholders to fill before production

These are intentionally placeholders — the values don't exist yet in the app repo:

1. **`assetlinks.json` → `sha256_cert_fingerprints`**: replace `REPLACE_WITH_APP_SIGNING_CERT_SHA256`
   with the SHA-256 of the app's **release** signing certificate:
   ```
   keytool -list -v -keystore <release.keystore> -alias <alias> | grep SHA256
   ```
   (The app currently still uses the template package `com.companyname.musicbox.app`;
   if that is finalized to a real id, update `package_name` here too.)
2. **`apple-app-site-association` → `appIDs`**: replace `TEAMID` with your Apple
   Developer Team ID (iOS is Phase 7/8 — not built yet). Format: `<TeamID>.<bundleId>`.

## GitHub Pages setup

Enable once (needs repo admin — Settings → Pages, or API):

```
gh api -X POST repos/daniel-mansson/tunetap-site/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
```

The `CNAME` file sets the custom domain automatically. After DNS verifies,
turn on **Enforce HTTPS** in Settings → Pages.

### DNS records (at your registrar for tunetap.to)

Apex `A` records → GitHub Pages:
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
Optional `AAAA` (IPv6):
```
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```

### Verify after deploy
```
curl -sI https://tunetap.to/.well-known/apple-app-site-association
curl -s  https://tunetap.to/.well-known/assetlinks.json
```
Note: GitHub Pages serves the extensionless AASA as `application/octet-stream`,
not `application/json`. Apple's iOS-14+ CDN tolerates this in practice; if AASA
verification ever fails, this content-type is the first thing to check (a host
that lets you set response headers — e.g. Cloudflare Pages — would fix it).
