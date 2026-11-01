# DevArt Consent — Runtime Limitations and Developer API

## Static HTML Protection

DevArt Consent blocks third-party scripts and embeds that are present in the
final HTML rendered by Joomla before the response leaves PHP. Blocked markup is
sent as cache-safe placeholders or `type="text/plain"` scripts and is restored in
the browser only after the relevant consent category is granted.

This model is intentionally compatible with full-page cache and Cloudflare:
the server does not read the consent cookie and does not vary HTML by visitor.

### Script type gate

Only executable script types are rewritten:

- missing `type`
- `text/javascript` / `application/javascript`
- `text/ecmascript` / `application/ecmascript`
- `module`

Non-executable types such as `application/ld+json`, `application/json`, and
`importmap` are left unchanged.

### PCRE fail-open

HTML rewriting uses `preg_replace_callback`. If a PCRE error returns `null`, the
plugin logs once per request (context and HTML length only) and returns the
original markup for that pass. Blocking remains fail-open so a regex failure
cannot blank the page.

## Cloudflare Rocket Loader

Frontend bootstrap and `consent.min.js` are enqueued with `data-cfasync="false"`
so Cloudflare Rocket Loader does not defer or reorder consent critical scripts.
Keep Rocket Loader enabled site-wide if needed; DevArt Consent opts itself out.

For Cloudflare IP hashing, Edge TTL versus the 14-day registry token window,
FPC purge after HTML-affecting saves, `com_ajax` bypass/rate limits, and
Dashboard misconfiguration hints, see [`cloudflare.md`](cloudflare.md).

## Dynamic Embeds (Scoped Runtime Observer)

JavaScript that creates or injects embeds after page load is intercepted by a
scoped, throttled `MutationObserver` when blocking layers are enabled in
settings. The observer watches only newly added `script` and `iframe` nodes with
a `src` attribute and applies the same category rules as the server-side
pipeline where possible.

Limitations remain:

- **Best-effort observer:** dynamic interception improves coverage for injected
  `script` / `iframe` nodes but is not a hard sandbox. Prefer
  `whenGranted()` or CSP for strict guarantees.
- **Pre-consent network race:** a dynamically inserted `iframe` or `script` with
  an immediate `src` may start a browser network request before the observer
  callback removes or rewrites the node. This is a browser timing limitation, not
  a server-side block. Use `whenGranted()` for critical widgets, or enforce
  `Content-Security-Policy` at the network level for strict guarantees.
- Inline scripts without a detectable provider signature are not matched
- Direct `script` / `iframe` nodes are inspected synchronously; container
  subtrees use a throttled scan (150 ms, timer not reset) that accumulates
  nodes in a `Set` so rapid successive batches are not dropped and continuous
  mutations cannot starve inspection
- When every enabled optional category is granted, the observer disconnects to
  reduce MutationObserver cost; it restarts after revoke or a partial choice
- `srcdoc`, lazy `data-src`, and attribute mutations after insertion may be missed
- The observer cannot undo scripts that executed before interception

For custom widgets, site or template code should still use
`window.DevArtConsent.whenGranted()` when automatic detection is insufficient.

`window.DevArtConsent.capabilities.dynamicEmbedObserver` is `true` when runtime
observation is active for the current page configuration.

## Developer API

`window.DevArtConsent` is available after the deferred runtime script loads.

| Method | Description |
|--------|-------------|
| `has(category)` | Returns whether a category is currently granted. |
| `isCategoryEnabled(category)` | Returns whether the category is enabled in component settings. |
| `isGranted(category)` | Alias of `has(category)`. |
| `whenGranted(category, callback)` | Runs `callback(state)` immediately when consent exists, and again after future grants. Returns an unsubscribe function. |
| `getState()` | Returns the current consent state object or `null`. |
| `acceptAll()`, `rejectAll()`, `save(categories)`, `revoke()` | User-facing consent actions. |
| `on(event, callback)` | Subscribe to `ready`, `change`, or `revoke`. |
| `capabilities.staticHtmlBlocking` | `true` — server-side HTML rewriting is active. |
| `capabilities.dynamicEmbedObserver` | `true` when runtime observation is active for the current page configuration. |

### Example: gate a custom widget

```javascript
document.addEventListener('DOMContentLoaded', () => {
  const loadMap = () => {
    const iframe = document.createElement('iframe');
    iframe.src = 'https://www.google.com/maps/embed?...';
    document.getElementById('map-slot').replaceChildren(iframe);
  };

  if (!window.DevArtConsent) {
    return;
  }

  window.DevArtConsent.whenGranted('external', () => loadMap());
});
```

## Policy URLs

Privacy and cookie policy links in the banner accept only:

- `http://` and `https://` absolute URLs
- site-relative paths starting with `/` (resolved with the Joomla base path, e.g.
  `/privacy` on `/demo3/` becomes `https://example.com/demo3/privacy`)

Schemes such as `javascript:` and `data:` are rejected on save and at render
time.

## Cache and Policy Version

The runtime configuration embeds `consent_version` in the cached HTML. After an
administrator increases the policy version, reverse proxies and full-page caches
must be purged so visitors receive the updated configuration. Until purge,
visitors may continue to see the previous embedded version and will not be
re-prompted even if their browser cookie is older.

The administrator UI shows a warning when the policy version is increased on
save. Joomla page cache groups are cleared automatically; reverse proxy and CDN
purge remain manual. See `COM_DEVARTCONSENT_CACHE_NOTE` and
`COM_DEVARTCONSENT_CACHE_POLICY_NOTE` in Settings.

When the consent registry is enabled, the same cached HTML also embeds a daily
rotating registry token (accepted for about 14 days). Purge CDN/page cache after
changing the Joomla secret or other settings that affect the embedded runtime
config.

## Consent Registry

- Visitor keys hash a validated IP (`REMOTE_ADDR` or optional `CF-Connecting-IP`)
  plus user agent. Raw IPs are never stored. Do not trust raw `X-Forwarded-For`.
  When using `CF-Connecting-IP`, the origin must accept traffic only from
  Cloudflare; otherwise the header can be spoofed by direct origin clients.
  If the public site is behind Cloudflare while Options still use `REMOTE_ADDR`,
  many visitors can share one visitor hash (Cloudflare edge IP) and hit the
  per-visitor rate limit early; the administrator dashboard warns when this
  mismatch is detected on the current admin request.
- A WAF / edge rate limit on the public `com_ajax` registry endpoint is required
  for serious protection. Built-in per-visitor and configurable global insert
  caps reduce abuse but can be saturated by spoofed User-Agent values. On very
  high-traffic consent spikes, the default global cap (300 inserts/minute) can
  also drop legitimate records — lower retention, raise the cap knowingly, or
  protect the endpoint at the edge.
- Retention purge deletes in batches of 5,000 with a soft ~20 second wall-clock
  budget and a hard batch ceiling per Scheduler run, so large backlogs may need
  more than one daily execution to catch up.
- CSV export streams in batches and stops at 10,000 filtered rows; a `truncated`
  marker row is written when that limit is hit.
- Retention cleanup uses `plg_task_devartconsentcleanup` with batched deletes.
  Install/update creates a daily enabled Scheduler task automatically when
  missing; administrators may disable that task intentionally.

## Runtime Performance

The blocking pipeline uses cheap string pre-checks before expensive regex passes
and skips entire layers when no matching markup is present. Runtime configuration
and JSON catalogues are cached once per request to avoid duplicate file reads
between `onBeforeCompileHead` and `onAfterRender`.

## Content Security Policy (CSP)

When the Joomla **System - HTTP Headers** plugin has nonce generation enabled,
DevArt Consent synchronises the application `csp_nonce` with the HTML document.
Inline bootstrap scripts, Google Consent Mode defaults, and appearance CSS are
emitted through Joomla document APIs so the renderers can attach the `nonce`
attribute automatically.

Sites without CSP nonces continue to work as before.
