# se-scann-dev.github.io

Controlled web fixtures for the [Usercentrics V3+ Scanner](https://github.com/AG-UC/scan-e2e)
regression suite. Each variant is a self-contained page designed to exercise a specific aspect of
scanner behaviour.

**Live**: <https://ag-uc.github.io/se-scann-dev.github.io/>
**Consumer**: <https://github.com/AG-UC/scan-e2e>

## Variants

| Path | Scope | Notes |
| --- | --- | --- |
| `/` | Landing page with nav | Minimal scripts; not a scan target. |
| `default/` | 10 commonly-deployed services across Essential / Functional / Marketing | Baseline regression target. |

Future variants will land in their own subdirectories — for example `/redirects/`,
`/late-tracker/`, `/query-param/`.

## Default-services fixture (`default/`)

| # | Service | Category | How it loads |
| --- | --- | --- | --- |
| 1 | Usercentrics CMP V3 (sandbox) | Essential | `<script src=…/ui/loader.js>` |
| 2 | Usercentrics Autoblocker | Essential | `<script src=…/autoblocker.js>` |
| 3 | Google reCAPTCHA | Essential | `<script src=…/recaptcha/api.js>` + image fixture |
| 4 | YouTube | Functional | `<iframe src=youtube.com/embed/…>` |
| 5 | Vimeo | Functional | `<iframe src=player.vimeo.com/video/…>` |
| 6 | Google Maps | Functional | `<iframe src=google.com/maps/embed?…>` |
| 7 | Google Analytics (GA4) | Marketing | `<script async src=googletagmanager.com/gtag/js?id=G-…>` |
| 8 | Google Tag Manager | Marketing | direct `gtm.js` loader |
| 9 | Hotjar | Marketing | direct `static.hotjar.com/c/hotjar-….js` loader |
| 10 | Facebook / Meta Pixel | Marketing | direct `connect.facebook.net/.../fbevents.js` loader |

All loaded as plain `<script src>` / `<iframe src>` / `<link>` tags — no `type="text/plain"` /
`data-usercentrics` consent gating. Services fire on page load and are directly observable by
the scanner.

## Conventions

- One subdirectory per scenario.
- Self-contained `index.html` per variant.
- Pages served from `main` on GitHub Pages — no build step. Changes go live within ~1 minute of
  merging.
- Settings ID `tPIg6i7JCCD-va` is the shared V3 sandbox configuration unless a scenario specifies
  otherwise.

## Adding a new variant

1. Create `<scenario-name>/index.html` — self-contained.
2. Document what scanner behaviour it exercises in the table above.
3. Add the corresponding manifest in [`scan-e2e/manifests/`](https://github.com/AG-UC/scan-e2e/tree/main/manifests).
4. Add a test that consumes the manifest in [`scan-e2e/tests/`](https://github.com/AG-UC/scan-e2e/tree/main/tests).

## Why this repo's name

The user-page URL pattern `<owner>.github.io` is only available if the repo is named exactly that
for a given account. This repo is hosted under `AG-UC` and named `se-scann-dev.github.io` for
naming consistency with the broader SE scanner ecosystem; it is served as a *project* page at
`https://ag-uc.github.io/se-scann-dev.github.io/` (not as a user page).
