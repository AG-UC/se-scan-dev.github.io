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
| `query-param/` | Minimal target that reflects `location.search` | Regression target for the scan-api `httpRequestInjections` feature (SE-3076). The suite injects query parameters and asserts they reach the scanned URL (`Pages.AllPages[].Url`). On its own start_url so injection runs don't share the throttle window with the default sanity scan. |
| `late-tracker/` | Pinterest Tag (`s.pinimg.com`) injected ~5s after load | Regression target verifying the scanner's delayed-activity observation window (≈10s per page) still records a late-firing tracker. Distinct from the two-pass / prior-consent mechanism. |
| `redirects/` (+ `redirects-target/`) | Client-side redirect to a page hosting Bing UET (`bat.bing.com`) | Regression target verifying the scanner follows a same-domain redirect and scans the destination. **Caveat:** client-side (meta-refresh + JS) redirect only — GitHub Pages cannot emit a real HTTP 301/302. |
| `gtm-ga/` | Google Analytics (`gtag/js?id=G-E2EVIAGTM1`) injected *via* Google Tag Manager (`gtm.js?id=GTM-E2EVIAGT`), not as a direct tag | Regression target for the "GA implemented via GTM" pattern. Asserts the GTM-injected GA request is still **observed**. **Scope:** the raw-blob oracle cannot prove GA-vs-GTM **attribution** (no service-name in the blob) — that is post-processing (scan-result-api / `template_id`) and lands with that oracle. **Caveat:** a static host can't run a server-configured GTM container, so a tiny shim performs the same runtime injection GTM would; the network fingerprint the scanner sees is identical. |
| `storage/` | Writes distinctive `localStorage`, `sessionStorage` and `IndexedDB` markers on load | Regression target (SE-3276) verifying the scanner records browser-storage writes in the result blob (captured as `StorageItem` Key/Value/Type and `IndexedDb` DatabaseName/operations). A short request burst keeps the observation window open for the async IndexedDB write. |
| `sitemap-crawl/` (+ `page-a/b/c/`, `sitemap.xml`) | Start page links to nothing; the three child pages are reachable only by parsing `sitemap.xml` | Regression target (SE-3275) verifying sitemap-driven discovery: the scan supplies the sitemap via the `sitemaps` config and asserts all three enumerated child URLs are crawled (not reachable by link-following). |
| `scan-result-smoke/` | Same 10 services as `default/` | Integration-tier target (SE-3224) for the scan-api -> scan-result-api cross-layer e2e: the suite scans it, bridges the result into scan-result-api via `POST /finished-scan`, and asserts each service maps 1-1 to a DPS `template_id` (incl. GA **and** GTM, catching a GTM-masks-GA regression). On its own start_url so it never shares `/default/`'s throttle bucket. |
| `scan-planner-smoke/` | Same 10 services as `default/` | Integration-tier target (SE-3243) for the scan-planner-triggered e2e: the suite fires scan-planner's `scan-now-by-domain-id` against the permanent dev fixture config whose domain points here, letting the REAL production pipeline (scan-planner -> scan-api -> Scanner Core -> webhook -> Pub/Sub -> scan-result-api) scan it, then asserts each service maps 1-1 to a DPS `template_id`. On its own start_url so it never shares a throttle bucket. |

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
