# Cookiebot YSC smoke — SE-3091 / CTS-4295

A superficial e2e smoke for the `scan-adapter` cookie-purpose cache-aliasing fix.

## Bug being verified

`CookieRepositoryPurposeCacheService` used to cache **mutable** response objects and consumers
rewrote `.Id` on every cache hit. On a hot cache (two trackers sharing a cache key in one scan,
or concurrent scans across domains hitting the same popular tracker), only the last `.Id`
assignment survived; the other tracker's purposes were silently lost. The fix swaps the cached
type to an `IReadOnlyDictionary<Guid, IReadOnlyList<CookieRepositoryPurposeEntry>>` — immutable
records, no shared mutable state.

## Why YSC

Per Michel Gammelgaard: dev Cookiebot's purpose DB is incomplete for several major providers
(GA, Meta, LinkedIn, HubSpot, DoubleClick), so blanks there don't distinguish "fix worked" from
"dev DB just doesn't have a description for that tracker". **YSC has purposes in both dev and
prod** — it's the cleanest single-tracker signal to smoke against.

## Smoke target

Live page: <https://ag-uc.github.io/se-scan-dev.github.io/cookiebot-smoke/>

Cookies the page produces when loaded by a Cookiebot scanner:

- `YSC` — youtube.com — Session — Statistics (this is the one we assert on)
- `VISITOR_INFO1_LIVE` — youtube.com — 6 months — Marketing (bonus signal; also has dev purpose)

## Recipe

```
1. Confirm dev scan-adapter is on the post-fix commit / restarted with the
   new build (cache is empty after restart — important for step 4 hot-cache
   variant to actually exercise the fix).

2. In Cookiebot dev:
   - Create / pick a test customer + CBID.
   - Set the start URL to:
       https://ag-uc.github.io/se-scan-dev.github.io/cookiebot-smoke/
   - Trigger a scan (force-crawl-scheduling flag, or manual scheduler endpoint).

3. Wait for the scan to reach a terminal state and for scan-adapter
   post-processing to finish (~5 min depending on environment).

4. Verify the cookie declaration. Two equally valid surfaces:

   (a) Public CD script — works once the dev CDN has refreshed:
       https://consent.cookiebot.com/<test-cbid>/cd.js
       (or the dev CDN host if that's different — Michel can confirm)
       The script renders the declaration when injected into a page; the
       simplest non-UI check is to grep the rendered HTML for the purpose
       string.

   (b) CB Admin / Manager UI:
       Open the test domain's cookie declaration view, language = English.
       Filter / find `YSC`. Confirm a non-empty purpose is rendered.

5. (Cross-scan stress, optional) Trigger a SECOND scan immediately after
   the first against a different test domain that ALSO embeds YouTube,
   without restarting scan-adapter. Both declarations must end up with
   YSC purpose populated. Pre-fix the second scan could lose purposes
   because the cache hit returned a `.Id`-mutated reference; post-fix
   the `IReadOnlyDictionary` makes the corruption physically impossible.
```

## Pass / fail in one line

```
PASS  →  cookie declaration for the test CBID lists YSC with a non-empty CookiePurposes field
FAIL  →  YSC purpose field is empty (= the bug is still live in this deploy)
```

The PR description is explicit that re-scanning is required after deploy to repopulate. A
fresh scan on this page is exactly that: a `force-crawl-scheduling` tick → fresh scan →
scan-adapter populates cache from cold for the first hit, then exercises the cache for the
second YSC observation in the same scan-result mapping pass.

## Out of scope for this smoke

- The full 8-language translation matrix (en / da / de / es / fr / it / nl / pt-BR). The PR
  fixes only the cache-aliasing bug. Missing translations / missing-Finnish-Swedish are
  separate Cookiebot-DB content concerns the SE-3091 ticket conflated.
- Other major providers (GA, Meta, LinkedIn, HubSpot, DoubleClick). Their dev-DB purposes are
  unreliable; verify those post-prod-redeploy by retesting `hintogroup.eu`'s public
  declaration at `https://consent.cookiebot.com/48df3734-a661-4879-b14f-87d29986aa06/cd.js`
  once a fresh scan has flowed through prod.
- Concurrent-scans race condition at high concurrency. The recipe above (step 5) covers the
  minimum cross-scan case. A real concurrency stress test would need ≥10 simultaneous scans
  on different domains all loading YouTube; out of scope for a smoke.
