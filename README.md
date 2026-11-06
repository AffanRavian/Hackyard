# ShopKhata

**Your khata, without the internet.**

ShopKhata is an offline-first credit ledger for Pakistani kiryana stores. Corner shops still keep customer *udhaar* in a paper register — the khata. When the lights go or mobile data drops (load-shedding, patchy coverage, the cost of a data pack), that paper book still works. ShopKhata does too. After the first load, every page, every rupee, and every reminder lives on the phone. Customer debts never leave the device. Reminders go out as ordinary SMS, which does not need mobile data.

Built for [Hackyard Yard #6: Offline](https://hackyard.tech/yards/yard-6).

## Features

- **First-run setup** — shop name, owner name, or a one-tap demo shop with eight realistic customers
- **Dashboard** — total to collect, weekly udhaar/payments, top dues, live online/offline badge
- **Customer list** — search, sort by dues / recent / A–Z, coloured avatars
- **Ledger** — gave udhaar / got payment, running balance, swipe or long-press to delete with undo
- **SMS reminder** — opens the phone SMS app with a bilingual template (`sms:` link, no internet)
- **Statement image** — drawn on a local `<canvas>`, download PNG or `navigator.share`
- **English / Urdu** — full UI strings, RTL, local Urdu font stack (never fetched)
- **Backup & restore** — JSON export/import with replace or merge, backup-age nudge
- **Optional 4-digit PIN** — SHA-256 on device via Web Crypto
- **Offline proof page** — network, service worker, cache file count, storage estimate, remote-request counter (should stay **0**)

## The airplane-mode test (60 seconds)

1. Build and serve the static files (`npm run build`, then `npx serve out`).
2. Open the app once while online so the service worker can precache every file.
3. Tap **Load demo shop** (or add a real customer).
4. Open DevTools → Network → **Offline**, or turn on airplane mode.
5. Hard reload (Ctrl+Shift+R / pull-to-refresh).
6. Visit Home, Customers, a ledger (`/customer/?id=…`), Proof, and Settings. Add an entry. Send an SMS reminder (the SMS app still opens).
7. On **Proof**, confirm: Offline: still working, service worker controlling the page, **0 remote requests**.

That is the whole product. If the hard reload still works, the khata works in a blackout.

## How the offline works

- **Next.js static export** (`output: "export"`) — the build is plain HTML/CSS/JS in `out/`. No API routes, no server actions, no SSR after the first download.
- **Hand-written service worker** (`public/sw.js`) — not next-pwa. A postbuild script walks `out/`, writes `precache-manifest.json`, and injects a versioned cache name plus the full URL list into `sw.js`.
- **Navigation with query strings** — `/customer/?id=abc` is matched with `ignoreSearch: true` so the cached `/customer/index.html` is served. There are no dynamic `/[id]` routes, because static export cannot pre-render them.
- **IndexedDB** — customers, entries, and settings stay on the device. Balance is always computed, never stored.
- **Zero runtime network** — no CDNs, no Google Fonts, no analytics. System fonts only. SMS uses the `sms:` URL scheme.
- **GitHub Pages** — `trailingSlash`, `basePath` from `NEXT_PUBLIC_BASE_PATH`, and `.nojekyll` so `_next/` is served.

## Screenshots

_Add screenshots here before the demo:_

- Dashboard (total to collect)
- Customer ledger
- Offline proof page (0 remote requests)
- Urdu / RTL view

## Demo video

_Add your demo video link here._

## Run locally

```bash
npm install
npm run dev
```

The service worker **only registers in production**. `next dev` will look online even though IndexedDB still works. To test the real airplane-mode path:

```bash
npm run build
npx serve out
```

Open http://localhost:3000 (or the port `serve` prints). Load once, then go offline and hard reload.

To mimic GitHub Pages locally:

```bash
# PowerShell
$env:NEXT_PUBLIC_BASE_PATH="/shopkhata"; npm run build
npx serve out
```

Then visit `http://localhost:3000/shopkhata/`.

## Deploy to GitHub Pages

1. Create a GitHub repo and push this project (default branch `main`).
2. Repo **Settings → Pages → Source: GitHub Actions**.
3. The workflow at `.github/workflows/deploy.yml` builds with `NEXT_PUBLIC_BASE_PATH=/<repo-name>` and deploys `out/`.
4. After the first successful run, the app is at `https://<user>.github.io/<repo-name>/`.

If the repository is named `shopkhata`, the public URL is `/shopkhata/`. Change nothing — the workflow reads `github.event.repository.name`.

## Project layout

```
app/           pages (static routes only; customer ledger uses ?id=)
components/    UI
context/       IndexedDB-backed app state
lib/           db, i18n, SMS, statement canvas, backup, seed
public/sw.js   service worker (placeholders filled in postbuild)
scripts/       precache + icon generators
```

## License

MIT — see [LICENSE](LICENSE).
