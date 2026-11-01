# DevArt Consent — Architecture

## Repository Layout

`source/` is the only development source:

- `source/package/` — outer Joomla package manifest and package language.
- `source/extensions/component/com_devartconsent/` — administrator component
  and shared media assets.
- `source/extensions/plugins/plg_system_devartconsent/` — system plugin and
  frontend injection source.

Generated packages belong only under `builds/`.

## Package Composition

The package manifest declares three inner extensions:

- `com_devartconsent`
- `plg_system_devartconsent`
- `plg_task_devartconsentcleanup` (optional registry retention via Joomla Scheduler)

Frontend media prefers minified `consent.min.js` / `consent.min.css` via WebAssetManager
(`version=auto`). Edit the non-min sources and regenerate with
`node scripts/minify-assets.mjs`. External provider domains live in
`media/data/providers.json`.

The build tooling derives the package name, version, inner archive names, and
inner archive folder from the package manifest.

## Administrator

The component provides Joomla administrator views for:

- Dashboard (configuration and protection status summary)
- Settings
- Tools
- Consents

Settings are stored as component parameters and include the banner, consent
categories, policies, appearance, external embeds, Google services, marketing
pixels, custom blocking rules, Google Consent Mode, and optional consent
registry configuration.

The consents view lists server-side registry records when registry recording is
enabled. Filters and CSV export apply to the active filter set.

## Frontend Runtime

The system plugin subscribes to Joomla's head-compilation and post-render events.
It loads the component media assets, constructs a cache-neutral runtime
configuration, injects the banner, blocks protected third-party scripts and
embeds in the rendered HTML, and restores them only after consent.

The browser runtime:

- Reads and validates consent from a first-party cookie
- Supports accept all, reject all, custom preferences, and revocation
- Exposes `window.DevArtConsent` and consent lifecycle events
- Documents runtime limits and the developer API in `docs/limitations.md`
- Updates Google Consent Mode when enabled
- Activates blocked scripts and embeds after the relevant category is granted
- Deactivates protected scripts when consent is withdrawn
- Restores blocked external content after external-content consent

Frontend banner text is resolved from `frontend-messages.json` for the active
site language (`en-GB` or `el-GR`), with plugin language files as fallback.

## Blocking Pipeline

Server-side blocking runs in this order on the rendered HTML:

1. Google services (when enabled per service)
2. Marketing pixels (when enabled per service)
3. External embeds (when external protection is enabled)
4. Custom administrator rules (when enabled)

Built-in detectors always run before custom rules. Each blocked element is sent
as cache-safe `type="text/plain"` markup and activated in JavaScript only after
the required consent category is granted.

DevArt Consent bootstrap and media assets are excluded from script blocking.

## Google Services

Each Google service has an ON/OFF switcher. When enabled, matching scripts are
auto-detected in the rendered HTML, converted to blocked placeholders, and
activated only after the relevant consent category is granted. No IDs or template
changes are required.

Category mapping:

- GA4 — analytics
- GTM — analytics or advertising
- Google Ads — advertising
- AdSense — advertising
- Google Ad Manager (GPT) — advertising

## Marketing Pixels

Each marketing service has an ON/OFF switcher with the same auto-detect model as
Google services. Supported detectors include Meta Pixel, TikTok Pixel, and
LinkedIn Insight Tag. All map to the advertising category.

Marketing pixel blocking targets tracking scripts and images. Embedded social
content is handled separately under External embeds.

## Custom Blocking Rules

Administrators can define up to 30 custom rules. Each rule specifies:

- Match target: script URL, inline script code, or iframe source
- Contains pattern: case-insensitive substring match
- Consent category: analytics, advertising, or external

Custom rules run after all built-in detectors.

## External Content

Embed protection is independently configurable for YouTube, Vimeo, Google Maps,
Facebook, Instagram, TikTok, X/Twitter, OpenStreetMap, SoundCloud, and Spotify.
Enabled iframe and supported script-based embeds are transformed into
placeholders before the response reaches the browser.

External embed blocking is separate from marketing pixel blocking. For example,
TikTok video embeds require external-content consent; TikTok Pixel requires
advertising consent.

## Appearance

Appearance settings control DevArt theme colours, corner radius, and the
floating privacy-settings button icon. Theme CSS variables are injected into the
page; the floating button uses the active theme primary colour.

## Consent Registry

When enabled, consent decisions can be recorded server-side through a
`com_ajax` endpoint. Recording is optional and off by default. Visitor
identification uses a hashed visitor key; no Joomla session is required for
frontend consent.

Registry hardening notes:

- Visitor hashing uses a validated IP from `REMOTE_ADDR` or, when configured,
  `CF-Connecting-IP` only (never raw `X-Forwarded-For`). Use the Cloudflare
  source only when the origin accepts traffic from Cloudflare alone; otherwise
  a direct-to-origin client can spoof `CF-Connecting-IP`.
- The registry AJAX token is a daily rotating HMAC with a 14-day acceptance
  window so full-page cache can keep serving an older embedded token.
- Same-site checks compare the request Origin/Referer **host** only.
- Duplicate `(consent_id, revision)` inserts rely on the existing UNIQUE key
  (MySQL 1062) instead of a pre-insert existence query.
- Payload validation runs before rate-limit COUNTs. Per-visitor and configurable
  global insert caps limit abuse; a WAF / edge rate-limit on the `com_ajax`
  registry endpoint is required for serious protection on public sites (caps
  alone are not sufficient against UA spoofing).
- Dashboard registry totals use a short-lived cache key that is removed on write
  or purge without wiping the whole `com_devartconsent` cache group.
- Rate-limited registry rejects are counted per UTC day (cache-backed) and shown
  on the dashboard when non-zero.
- Frontend `providerCatalogue` is built from `media/data/providers.json` so PHP
  and JS share one provider domain/label source.
- Server-side HTML blocking is orchestrated by `HtmlBlocker` with dedicated
  Google/Marketing layers and shared `HtmlBlockingSupport` helpers.
- Retention deletes are performed by `plg_task_devartconsentcleanup` in
  batched `DELETE … LIMIT 5000` loops (cap per run) using the Options retention
  setting. Install/update seeds a daily Scheduler task when missing; an
  existing task state is left unchanged.
- After changing the Joomla secret, registry token settings that affect cached
  HTML, or policy version, purge page cache and CDN/reverse-proxy cache.

## Cache Strategy

The plugin sends the same blocked HTML and banner configuration to every
visitor. Consent is read and applied in the browser, without a Joomla session,
per-visitor database writes for banner display, or `Vary: Cookie`. This
preserves page-cache and reverse-proxy compatibility, including full-page cache
and Cloudflare.

When the policy version changes, administrators must purge cached HTML so the
new `consent_version` reaches visitors. The settings save flow clears Joomla
`page`, `com_content`, and `_system` cache groups automatically when the policy
version increases, and reminds administrators to purge reverse proxies and CDNs.

Per request, the plugin caches its runtime configuration and reuses JSON
catalogue reads. The blocking pipeline applies cheap pre-checks before regex
rewriting and skips layers that cannot match the current HTML.

See `docs/limitations.md` for dynamic embed limits and the public
`window.DevArtConsent` API.

## Build and Validation

- `scripts/validate.php` validates manifests, references, versions, XML, JSON,
  PHP syntax, and the PHP minimum.
- `scripts/build.php` creates the declared inner extension ZIPs, the outer
  package, and its SHA-256 checksum.
- Build tooling does not modify `source/` and does not overwrite an existing
  build.
