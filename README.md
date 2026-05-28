# se-scan-dev.github.io

Static test pages for the Usercentrics V3+ Scanner regression suite. Each variant is a
self-contained page designed to exercise a specific aspect of scanner behaviour.

**Live**: <https://ag-uc.github.io/se-scan-dev.github.io/>
**Consumer (scan-e2e-tests suite)**: <https://bitbucket.org/usercentricscode/scan-e2e-tests>
(planned canonical location alongside other SE repos)

## Variants

| Path | Scope | Notes |
| --- | --- | --- |
| `/` | Landing page with nav | Minimal scripts; not a scan target. |
| `default/` | 10 commonly-deployed services across Essential / Functional / Marketing | Baseline regression target. |
| `missing-one/` | Same 10 services with LinkedIn Insight Tag intentionally absent | Threshold-tolerance target. The consuming scenario still expects all 10 services with a 90% threshold — so 9/10 passes and the missing service is surfaced in the detection report for manual verification. |
| `cookiebot-smoke/` | YouTube-only page (sets `YSC` + `VISITOR_INFO1_LIVE`) | Smoke target for the **Cookiebot pipeline**, specifically the [SE-3091 / CTS-4295](https://usercentrics.atlassian.net/browse/SE-3091) `scan-adapter` cookie-purpose cache-aliasing fix. Configure as the start URL of a Cookiebot dev test customer; verify `YSC` has a non-empty `CookiePurposes` after scan. See [`cookiebot-smoke/README.md`](./cookiebot-smoke/README.md). |

Future variants will land in their own subdirectories — for example `/redirects/`,
`/late-tracker/`, `/query-param/`.

## Default-services test page (`default/`)

| # | Service | Category | How it loads |
| --- | --- | --- | --- |
| 1 | Usercentrics CMP V3 (sandbox) | Essential | `<script src=…/ui/loader.js>` |
| 2 | Google reCAPTCHA | Essential | `<script src=…/recaptcha/api.js>` + image |
| 3 | YouTube | Functional | `<iframe src=youtube.com/embed/…>` |
| 4 | Vimeo | Functional | `<iframe src=player.vimeo.com/video/…>` |
| 5 | Google Maps | Functional | `<iframe src=google.com/maps/embed?…>` |
| 6 | Google Analytics (GA4) | Marketing | `<script async src=googletagmanager.com/gtag/js?id=G-…>` |
| 7 | Google Tag Manager | Marketing | direct `gtm.js` loader |
| 8 | Hotjar | Marketing | direct `static.hotjar.com/c/hotjar-….js` loader |
| 9 | Facebook / Meta Pixel | Marketing | direct `connect.facebook.net/.../fbevents.js` loader |
| 10 | LinkedIn Insight Tag | Marketing | direct `snap.licdn.com/li.lms-analytics/insight.min.js` loader |

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
3. Add the corresponding scenario YAML in the `scan-e2e-tests` Bitbucket repo
   (`tests/scenarios/`).
4. Add a spec that consumes the scenario in `scan-e2e-tests` (`tests/sanity/` or
   `tests/regression/`).
