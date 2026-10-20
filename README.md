# What Happens When You Type a URL

**An interactive, scroll-driven 3D journey of a single web request.**

Type a URL. In about **1.2 seconds** your request checks the browser and OS caches, gets resolved by recursive DNS, completes a TCP handshake and a TLS 1.3 negotiation, lands on a CDN edge in Frankfurt — misses the cache — crosses the Atlantic on the real **Dunant submarine cable** to an origin in Ashburn, and comes back as HTML that becomes a DOM, a CSSOM and finally the page on your screen.

This site lets you *watch all of that happen*, in 3D, stage by stage. Just scroll.

![Stage 5: the packet on the Dunant cable](public/og.jpg)

## Trace any site — live

Click the address bar (or just start typing) and enter any URL, or share a link like `/?url=github.com`. Every stage then uses that site's real data:

- **DNS:** A record, TTL, authoritative nameserver, and the TLD's servers (`/api/trace`).
- **TLS:** protocol, negotiated cipher and certificate chain, from a real handshake (`/api/trace`).
- **CDN edge:** provider, the point of presence near *you*, and the cache status, read from response headers by `/api/probe`. It runs in five regions (Frankfurt, Washington, San Francisco, Mumbai, Singapore) and each visitor is routed to the nearest, so the PoP it reaches is close to the one their own browser uses. Vercel, Cloudflare, CloudFront, Fastly, Netlify and Akamai are recognised; the raw headers are shown in the edge stage's info card.
- **Origin:** placed on the globe when the provider reveals it; otherwise the map says so instead of guessing.
- **Timing:** the elapsed counter is an estimate from great-circle distances (light in fibre ≈ 200 km/ms, ×1.4 for real routes), marked `≈`.

Safety: only public hostnames are accepted; every resolved address must be public; the TLS check is pinned to the IP that was validated; the probe sends one GET to `/` without following redirects; everything times out within 5 s.

The landing page plays a scripted example run to **example.com** (labelled "example run" in the HUD).

## The journey — 7 stages

| # | Stage | What happens |
| --- | --- | --- |
| 01 | The URL bar | Cache hierarchy: browser cache → OS cache, before any network call |
| 02 | DNS | Recursive resolution: resolver → root → TLD → authoritative server |
| 03 | TCP handshake | SYN, SYN-ACK, ACK — three packets before a single byte of data |
| 04 | TLS 1.3 | One round trip: cipher agreement, certificate chain, shared secret |
| 05 | The CDN edge | Anycast to the nearest PoP — and this one is a cache **MISS** |
| 06 | Under the ocean | ~6,400 km of fibre on the seabed, Frankfurt → Ashburn via the Dunant cable |
| 07 | Response & render | HTML streams back; DOM + CSSOM grow; the page paints |

The edge comes before the ocean on purpose: anycast sends you to the nearest edge first, and only a cache miss crosses the ocean. Coordinates and latencies are approximate and marked as such in `src/lib/journey.ts`.

## Made for every screen

- Fully responsive HUD: a floating glass stage-dock and large touch targets on phones, a cinematic rail and gradient typography on desktop.
- Adaptive quality: the renderer detects your device tier and scales resolution and effects to stay smooth.
- Renders on demand — when you stop scrolling, the GPU stops working.

## Stack

Next.js 15 (App Router, TypeScript, Turbopack) · React Three Fiber · drei · GSAP ScrollTrigger · zustand

Nothing else 3D: bloom is three's own `UnrealBloomPass`. The globe, cables, towers, packet and every label are procedural. The one modelled asset is the TLS padlock.

## Performance

Full methodology and per-stage tables are in [PERF.md](PERF.md).

| | Before | After |
| --- | --- | --- |
| Padlock GLB | 67.0 MB, 1.95 M triangles | **295 KB**, 20 k triangles (−99.6%) |
| JS a visitor downloads (gzip) | 529 KB | **474 KB** |
| Everything transferred, full journey | ~1.34 MB | **~0.83 MB** |
| Draw calls, DNS / TCP / render stages | 65 / 68 / 97 | **43 / 43 / 67** |
| Main-thread work before the first frame | 154 ms | **26 ms** |

Every stage renders at 5–7 ms/frame on an M1 Pro at 1792×1344 with bloom (140–200 fps-equivalent).

## Run it locally

```bash
npm install
npm run dev
```

Open http://localhost:3000 and scroll.

| Flag | What it does |
| --- | --- |
| `?debug` | fps / draw calls / triangles / memory readout, leva panel, `window.__bench` |
| `?tier=low` / `?tier=high` | force the quality tier (auto-detected otherwise) |
| `?geo=0` | disable the geolocated starting city |
| `?video` | recording mode used by `npm run record` |

### Record the shareable video

```bash
npm run build && npm start
npm run record                                              # full cut, 16:9
npm run record -- --cut short --width 1080 --height 1350    # ~40 s cut, 4:5
```

Writes `video/journey-<cut>-<w>x<h>.mp4` (H.264 1080p60 + AAC). Each video opens cold on the globe shot under a hook line, hard-cuts to typing, and carries an original soundtrack synthesised in `scripts/soundtrack.mjs` — no samples, nothing to license.

## How it's built

- `STORYBOARD.md` is the script; `design/` holds the reference stills each stage was matched against.
- `SCENES.md` documents the stage interface: each stage is one file exporting `{ Scene, cameraIn, cameraOut, camera?, packet?, duration, models? }`.
- `src/lib/stages.ts` is the registry: copy, hotspots, fog, the elapsed-ms budget and scroll ranges.
- `src/core/` holds the runtime: camera rig, hero packet, stage lifecycle, shared world, bloom, the demand-frame driver and adaptive resolution.

## Assets

- `design/*.png`: reference stills (hero, s1–s6, s5 4:5 crop).
- `public/models/padlock.min.glb`: the only model (meshopt + WebP).
- `public/og.jpg`: the share image, cropped from `design/s5.png`.

---

**Asaad Siddiqui**
