# DevArt Consent — Cloudflare and Full-Page Cache

DevArt Consent is designed so the origin HTML does not vary by visitor consent.
The banner and blocked placeholders are the same for every first response; the
browser applies consent after load. That model is compatible with Cloudflare
Full Page Cache (FPC) and common Joomla page caches.

## Rocket Loader

Frontend bootstrap and `consent.min.js` ship with `data-cfasync="false"`.
Rocket Loader may stay enabled site-wide; consent-critical scripts opt out so
they are not deferred or reordered.

## Visitor IP for the consent registry

Registry Options can hash visitor IP from:

| Mode | Source | When to use |
|------|--------|-------------|
| Default | `REMOTE_ADDR` | Origin receives direct client connections |
| Cloudflare | validated `CF-Connecting-IP` | Origin accepts traffic **only** from Cloudflare |

Do **not** enable Cloudflare IP mode if the origin is reachable bypassing
Cloudflare. Untrusted clients could spoof `CF-Connecting-IP`.

The Dashboard warns when registry mode is `REMOTE_ADDR` but the current admin
request presents a different validated Cloudflare connecting IP (typical
misconfiguration signal behind Cloudflare).

## Full-page cache expectations

- Server-side code does **not** read the consent cookie to choose HTML.
- Changing HTML-affecting settings (or raising `consent_version`) clears
  Joomla page HTML cache groups that may hold consent-sensitive markup.
- After policy changes behind Cloudflare FPC, **purge** the relevant Cloudflare
  cache (or wait for TTL) so visitors receive the new placeholders/banner and
  the current registry endpoint/token bootstrap.
- Prefer an Edge/HTML TTL **no longer than 14 days**. Registry AJAX accepts a
  daily-rotating HMAC token with a **14-day** acceptance window so cached pages
  can still record consent; HTML cached longer than that window can present a
  token the origin will reject until the page is purged or refreshed.
- After any HTML-affecting save (blocking toggles, messages, `consent_version`,
  appearance that changes injected markup), purge Cloudflare for the public
  site in addition to Joomla page-cache clear.

## Registry endpoint at the edge

- Bypass or Cache Level bypass the public registry `com_ajax` URL
  (`index.php?option=com_ajax&group=system&plugin=devartconsent&format=json`)
  so consent recording is never served from cache.
- Apply an edge/WAF rate limit on that endpoint. The component global insert
  cap (default **300/minute**, configurable in Options) is a safety net, not a
  substitute for edge protection; raise it only after measuring real peak
  traffic and rejection rates.

## Related

- Runtime limitations and developer API: [`limitations.md`](limitations.md)
- Registry Options text in the component configuration UI
