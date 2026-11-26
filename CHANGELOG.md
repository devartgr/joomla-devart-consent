# Changelog

## 1.1.9 — 2026-11-26

Major stability, security, and tooling release for Joomla 6 production sites.
Safe update from the public `1.1.8` baseline.

### Security
- Hardened client-side embed restore against untrusted markup (element/attribute allowlists; strips event handlers, `srcdoc`, and non-http(s) URLs)
- Embed placeholders reject the always-granted `necessary` category
- Fallback embed `data-src` requires a catalogue- or custom-rule-matched http(s) URL
- Visibility cookie wipe emits `devartconsent:revoke` when consent was previously present
- Tools same-site URL analyze is SSRF-safe (site host from `live_site` / site root, port checks, no redirects, download size cap)

### Blocking engine
- Shared `HtmlTagAttributes` / `HtmlRawTextGuard` helpers for exact attribute reading and raw-text protection
- Iframe/img/blockquote passes no longer rewrite markup that appears only inside JavaScript strings, comments, `style`, or `textarea`
- Quote-aware comment/tag masks; bare `a < b` before comments keeps comment bytes; attribute comment residuals closed
- Skipped `svg`/`template` segments unmasked before storage; self-closing exception applies to `<svg/>` only
- Lazy `about:blank` / `data:` / placeholder `src` prefers the rule-matched `data-src` candidate for storage and restore
- Detectors evaluate both `src` and `data-src` URL candidates (Google, marketing, custom, external)
- `type="module"` preserved as `data-devart-type` and restored on activation
- Mixed GA4 + Google Ads inline `gtag('config', …)` classification with Consent Mode aware semantics
- Runtime config exposes resolved protection rules so the dynamic observer mirrors PHP detectors
- Single-pass `strtr` token restore for large/comment-heavy HTML

### Consent UX and API
- Restore lazy-loaded embeds after consent (`data-src` promotion for iframes)
- Stricter category handling for restored embed markup
- Session “Allow content” embeds deactivate when the consent cookie disappears
- Allow content no longer opens preferences or pre-ticks category checkboxes
- Allow-content before first choice is session preview only; optional revoke page reload
- `whenGranted` fires again after silent cookie loss then re-accept
- Reject/revoke without prior activation no longer removes a neighbouring foreign `<script>` by adjacency
- Revoke clears known first-party tracker cookies on a conservative allowlist
- Early cookie hint + CSS suppress embed blocked-message flash when external consent is already granted
- `data-label` on external placeholders for stable revoke labels
- Theme primary colours meet WCAG AA with white button text
- Google / marketing prechecks cover inline signatures without matching bare Consent Mode bootstrap needles

### Administrator Tools and registry
- Tools → Blocker **Self-check** (built-in YouTube + gtag fixture scored against current Settings)
- Tools same-site URL analyze for frontend pages on this host
- Soft `registry_max_records` trim runs in the cleanup Scheduler task (not on public inserts)
- Configurable `registry_export_limit`; CSV export uses keyset pagination
- Dashboard / Settings warning when blocking is on but the category switch is off
- Registry dashboard `COUNT(*)` uses a forced short TTL and is no longer cleared on every insert
- Policy HTML cache purge groups are `page` and `_system` only

### Notes
- Install or update using the full `pkg_devartconsent` package only
- Purge full-page cache and CDN after updating so new `consent.js` attributes match cached HTML
- Italian (`it-IT`) curated locale from `1.1.8` is unchanged for reviewed banner/frontend strings; a few new administrator Tools/registry labels may still use English until a follow-up native review

## 1.1.8 — 2026-11-26

Substantial quality release: Italian native-speaker review, permanent Settings
message overrides, developer API reliability, and admin UX polish.

### Added

- Settings → Messages: permanent per-language frontend message overrides
  (component params; empty fields keep the built-in pack / plugin translation;
  package updates do not wipe site overrides)
- Message resolve order: Settings overrides → `frontend-messages.json` → plugin INI
- Collapsible Messages UI groups (Banner, Buttons, Categories, Placeholders,
  Policy link labels)

### Fixed

- Italian (`it-IT`) native-speaker review (community issue #2 / @mauroplg):
  plugin banner strings, component sys.ini, visible administrator strings, and
  the frontend messages pack
- Empty `COM_DEVARTCONSENT` menu title and broken `#` analytics label in it-IT
- `it-IT` marked curated/protected so generators no longer overwrite it
- Developer API: `on('change'|'ready'|'revoke')` now matches `emit()` full event
  names, so `whenGranted()` runs after a later accept (not only when consent
  already exists)
- Server-rendered embed placeholders use the same resolved messages as the banner
- Consent CSV export passes an explicit `fputcsv()` escape argument for PHP 8.4+
- Settings language lists for Messages/Policies use installed site language packs
  (not only published content languages)

### Changed

- Admin language notes clarify installed site packs vs content languages
- Machine-locale admin strings for the new Messages UI refreshed where they still
  matched English

## 1.1.7 — 2026-11-22

### Fixed

- Russian (`ru-RU`) reject-all button label: `Отвергнуть все` → `Отклонить все`
  (plugin language + frontend messages pack; community report)

## 1.1.6 — 2026-11-01

### Fixed

- Banner markup is injected before the last `</body>` only, so literal `</body>`
  text in page content is not rewritten
- Registry `sendBeacon` failures fall back to `fetch` so consent recording is not
  silently skipped when the beacon queue rejects the payload

### Changed

- Registry AJAX JSON body reads are capped at 4 KB before decode (FormData path
  unchanged)
- Settings save clamps `cookie_days` to 1–3650, requires `consent_version` ≥ 1,
  and whitelists banner `position` to `bottom|top|modal`
- Registry retention purge uses a ~20 second time budget with a hard batch
  ceiling instead of a fixed 40-batch limit
- Registry AJAX releases the Joomla session lock early (no session writes on
  that path)

### Added

- Dashboard warning when consent registry uses `REMOTE_ADDR` while the admin
  request shows a different validated Cloudflare `CF-Connecting-IP`
- Operator guide `docs/cloudflare.md` (Rocket Loader, CF IP mode, FPC)

### Fixed

- HTML blocker PCRE log context labels match the layer under rewrite (Google /
  Marketing / custom) instead of leftover cross-layer strings
- Banner dialog uses `aria-modal="false"` for bottom/top and `aria-modal="true"`
  with a Tab focus trap when `position=modal`

### Changed

- HtmlBlockingGuard PCRE warnings use the shared `plg_system_devartconsent`
  logger category (same as HtmlBlockingSupport)
- Consents admin task-reset script moves to WebAsset `addInlineScript`
- `declare(strict_types=1)` on remaining core PHP entry points (component,
  DisplayController, views, helpers, system plugin)
- Registry endpoint is root-relative (`Uri::root(true)` path) so multi-host /
  proxy setups post to the origin that served the page
- Dynamic embed container scan uses a 150 ms throttle (no timer reset) so
  continuous DOM mutations cannot starve deferred inspection
- `scripts/validate.php` fails when `consent.min.js` / `consent.min.css` are
  older than their sources (run `node scripts/minify-assets.mjs`)
- Machine-locale administrator strings that still matched English (including the
  Cloudflare registry dashboard warning and registry Options labels) are
  refreshed; technical IP-source option labels stay English by design
- Operator Cloudflare guide documents Edge TTL ≤14 days versus the registry
  token window, required FPC purge after HTML-affecting saves, and
  bypass/rate-limit guidance for the registry `com_ajax` endpoint

## 1.1.5 — 2026-10-03

### Fixed

- Script blocking applies only to executable script types; `application/ld+json`,
  `importmap`, and similar non-executable scripts are left unchanged
- PCRE rewrite failures are logged once per request and fail open with the
  original HTML
- Frontend dynamic observer debounce accumulates MutationObserver batches in a
  `Set` so rapid inserts are not dropped
- Embed placeholders use DOM/`textContent` instead of raw message `innerHTML`
- `reject_all` is no longer recorded as `accepted` when no optional categories
  are enabled
- Activated external scripts preserve original sync order (`async = false` when
  the source was not async)
- Package postflight enables the system plugin on install/discover_install only
  and no longer force-enables it on update
- Settings save clears Joomla page HTML cache groups when HTML-affecting fields
  change, not only when `consent_version` increases

### Added

- Registry Options: visitor IP source (`REMOTE_ADDR` or Cloudflare
  `CF-Connecting-IP`) and retention days
- Task plugin `plg_task_devartconsentcleanup` for scheduled registry retention
  cleanup via Joomla Scheduler (daily task auto-created on install/update)
- `providers.json` catalogue for external embed providers
- `HtmlBlocker` / `BannerRenderer` helpers extracted from the system plugin

### Security

- Missing component helpers no longer fatal the site; the system plugin returns
  early when the component is unavailable
- Registry visitor hashing never uses raw `X-Forwarded-For`; Cloudflare IP source
  accepts only a validated `CF-Connecting-IP` value
- Registry Options text warns that Cloudflare IP mode requires an origin that
  accepts traffic only from Cloudflare

### Changed

- Bootstrap inline script and consent assets enqueue set `data-cfasync="false"` for
  Cloudflare Rocket Loader compatibility
- Frontend assets load via WebAssetManager with `version=auto` and minified
  `consent.min.js` / `consent.min.css` (edit sources, run `node scripts/minify-assets.mjs`)
- Dynamic MutationObserver disconnects when all enabled optional categories are
  granted and restarts after revoke or a partial choice
- Settings save moved to `SettingsModel` with a thin `FormController`
- Dashboard registry total uses a short-lived cache invalidated on write/purge
- Registry AJAX token rotates daily (HMAC) with a 14-day acceptance window for
  full-page cache compatibility
- Same-site registry checks compare Origin/Referer host only
- Registry hot path drops pre-insert `revisionExists` / availability probes;
  duplicates rely on the existing UNIQUE key (MySQL 1062)
- Global insert rate cap added alongside the per-visitor limit
- Registry `record()` validates consent id / decision / source before rate-limit
  COUNTs, and uses one combined visitor+global COUNT query
- Registry retention purge deletes in `LIMIT 5000` batches with a per-run cap
- Added `scripts/run-registry-tests.php` for token, same-site host, IP source, and
  record/purge ordering checks (wired into `validate.php`)
- Replaced deprecated `DatabaseInterface::setQuery($query, $offset, $limit)` with
  `$query->setLimit()` in registry list/export paths
- Registry CSV export streams in 1,000-row batches and marks truncation at 10,000
- Settings save shows a CDN purge notice for any HTML-affecting change (not only
  policy version bumps) and treats `cookie_days` as HTML-affecting
- Registry dashboard total cache invalidates one key instead of cleaning the whole
  `com_devartconsent` group on every insert
- Global registry insert cap is configurable in Options (default 300/minute)
- Cookie expiry while a tab stays open now deactivates scripts/embeds via
  `applyState(null)`, not Consent Mode update alone
- Google blocking pre-check drops weak `dataLayer` / `gtag(` needles that matched
  the Consent Mode bootstrap on every page
- `HtmlBlockingGuard` logs PCRE mask failures once per request and uses a safer
  skip placeholder
- Frontend dynamic provider matching uses `providerCatalogue` from
  `providers.json` (single source of truth with PHP)
- HTML blocking split into `HtmlBlockingSupport`, `GoogleContentBlocker`, and
  `MarketingContentBlocker` under a thinner `HtmlBlocker` orchestrator
- Dashboard shows UTC-day rate-limited registry reject count when non-zero
- Frontend engine version `2.0.2`
- External embed blocking extracted to `ExternalContentBlocker`
- Administrator branding CSS/header injection moved to `AdministratorBrandingHelper`
- Single `EngineVersion` helper feeds plugin config, dashboard, and `consent.js`
- Real `consent.js` VM tests cover accept/reject/custom/revoke, policy mismatch,
  config `engineVersion`, and expired cookies (`consent-engine.test.mjs`)
- Frontend body processing uses a request-local flag instead of scanning HTML
  for `id="devart-consent"` (avoids false skips when page content mentions the id)
- Consent registry list/export ORDER BY adds `a.id` as a stable tie-break so
  OFFSET batch CSV export cannot duplicate or skip same-second rows
- Removed unused deprecated `ConsentsModel::getExportItems()`
- `validate.php` fails if `consent.js` engineVersion fallback drifts from
  `EngineVersion::VALUE`

## 1.1.4 — 2026-09-24

### Fixed

- Replaced deprecated `JPATH_COMPONENT_ADMINISTRATOR` with `JPATH_ADMINISTRATOR`
  for Settings form path and `__DIR__` for admin shared footers (Joomla 7 readiness)

## 1.1.3 — 2026-09-24

### Fixed

- Administrator Settings showed empty policy URLs after Apply/Save until Joomla
  cache was cleared; `_system` ComponentHelper params cache is now refreshed on
  every settings save
- Replaced deprecated `Factory::getCache()` with `CacheControllerFactoryInterface`
- Replaced deprecated Application `$input` access with `getInput()` in ConsentsModel

### Changed

- Replaced `Factory::getDate()` with `Joomla\CMS\Date\Date` in ConsentRegistryService
  and ConsentsModel

## 1.1.2 — 2026-09-18

### Fixed

- Multilingual policy URL saves now persist for Greek and other non-default languages
- Policies language selector stays on the last edited language after Apply/Save
- Flaky English policy URL display after save (nested Registry → single JSON map)

### Changed

- Settings → Policies uses one Privacy/Cookies editor pair with a hidden JSON language map

## 1.0.7 — 2026-08-27

### Fixed

- Dashboard hubs now load dedicated `admin.css` via `registerAndUseStyle`
  (same path as DevArt Business), restoring coloured circular icons and
  matching hub buttons

## 1.0.6 — 2026-08-27

### Fixed

- Dashboard hubs now match other DevArt dashboards: coloured circular icons,
  matching hub button colours, and a centred Options button

## 1.0.5 — 2026-08-27

### Fixed

- Duplicate administrator dashboard banner (template header + plugin submenu
  header injection)

## 1.0.4 — 2026-08-27

### Added

- Administrator dashboard hubs for Tools, Consents and Settings with distinct
  icons and colours
- Non-hub Options button under the hubs linking to Joomla component Options

### Changed

- Dashboard layout aligned with other DevArt hub dashboards (banner, intro,
  hubs, Options, status info)

## 1.0.3 — 2026-08-27

### Fixed

- Activated embeds no longer stretch to full container width with a collapsed
  height after removing the grey wrapper background

### Changed

- Activated iframes keep original width/height attributes; only `max-width: 100%`
  prevents overflow
- Activated wrappers keep the placeholder `min-height` reservation

## 1.0.2 — 2026-08-27

### Fixed

- Activated embed wrappers no longer keep a grey background or border after
  consent is granted
- Activated embed containers clear the placeholder `min-height` once the iframe
  is restored

### Changed

- Embed placeholder frame styling applies only while external content is blocked
- Appearance theme CSS variables apply to blocked embeds only, not activated
  wrappers
- `scripts/validate.php` allows historical `changelog.xml` entries and requires
  a current-version entry (aligned with DevArt Slider)

## 1.0.1 — 2026-08-10

### Added

- Synchronous inspection for directly added `script` and `iframe` nodes in the
  dynamic embed observer; debounced subtree scan only for containers
- `HtmlBlockingGuard` skips `<template>` and `<svg>` segments during server-side
  blocking transforms
- `qa/csp-verification.md` runtime CSP nonce checklist
- Node regression tests for observer scheduling (`consent-observer-schedule.test.mjs`)

### Changed

- Consent registry availability check uses a zero-row probe instead of
  `COUNT(*)`
- `GoogleServicesHelper` no longer reads alpha-era `*_mode` parameters
- `PackageVersionHelper` replaces suppressed XML loading in the dashboard
- Frontend consent engine `2.0.1`

## 1.0.0 — 2026-08-10

### Added

- First stable release after beta (`1.0.0-beta.1`–`beta.3`) and RC (`1.0.0-rc.1`)
  verification
- `qa/stable-signoff.md` production rollout checklist
- `docs/stable-release.md` stable scope and upgrade notes

### Fixed

- Administrator dashboard engine version now matches runtime engine `2.0.0`

## 1.0.0-rc.1 — 2026-08-10

### Added

- `SiteCacheHelper` clears Joomla `page`, `com_content`, and `_system` cache groups
  when the policy version increases on save
- `qa/rc-signoff.md` RC verification checklist
- Runtime matrix sign-off baseline with beta-cycle results recorded

### Changed

- Policy version save flow shows Joomla cache success, partial, or manual-clear
  messages plus a CDN/reverse-proxy reminder
- Documentation updated for automatic Joomla cache clear on policy bump

## 1.0.0-beta.3 — 2026-08-10

### Added

- Scoped, throttled `MutationObserver` for dynamically inserted `iframe` and
  `script` nodes when blocking layers are enabled
- `dynamicBlocking` runtime configuration and `capabilities.dynamicEmbedObserver`

### Changed

- Restored third-party scripts are tracked with an in-memory `WeakMap` instead
  of `data-devart-consent-runtime` on live script tags (fixes AdSense console
  warning)
- Consent registry table availability is cached per service instance
- Frontend consent engine version `2.0.0`

## 1.0.0-beta.2 — 2026-08-10

### Added

- `CspNonceHelper` synchronises Joomla `csp_nonce` with inline scripts and styles
- Administrator note in Settings about dynamic embed limitations
- Corpus policy URL vectors for parent-directory traversal attempts

### Changed

- Appearance CSS is emitted with `addStyleDeclaration()` so CSP nonces apply
- `PolicyUrlHelper` rejects relative paths containing `..`
- Package installer script uses `DatabaseInterface::class`

### Not included (planned for a later beta)

- Scoped MutationObserver for JavaScript-created embeds
- Regex performance consolidation

## 1.0.0-beta.1 — 2026-08-10

### Added

- First beta candidate release line after audit hardening (`0.1.42`–`0.1.46`)
- `qa/beta-signoff.md` cache-focused beta verification checklist
- `docs/beta-release.md` beta scope, limitations, and path to stable `1.0.0`

### Changed

- Update server metadata tagged as `beta`

## 0.1.46 — 2026-08-10

### Added

- Runtime verification corpus (`qa/corpus/`) with HTML and response fixtures
- `scripts/run-corpus-tests.php` integrated into `scripts/validate.php`
- `qa/runtime-matrix.md` for cache, CDN, template, and conflict scenarios
- `docs/re-audit.md` internal production-readiness summary after audit fixes

### Changed

- `RenderedBodyGuard` helper extracted from the system plugin for testable
  non-HTML response filtering

## 0.1.45 — 2026-08-10

### Changed

- P1 performance: per-request runtime config cache; cheap pre-checks before
  blocking regex passes; skip entire blocking layers when markup cannot match
- P1 performance: in-request cache for appearance and frontend-messages JSON
  catalogues
- Policy version save warning and stronger cache-purge guidance in Settings

## 0.1.44 — 2026-08-10

### Fixed

- Restored the `Joomla\CMS\Event\Plugin\AjaxEvent` import for the consent registry
  `com_ajax` endpoint

## 0.1.43 — 2026-08-10

### Fixed

- Site-relative privacy and cookie policy URLs now include the Joomla base path on
  subdirectory installs (for example `/privacy` on `/demo3/`).

## 0.1.42 — 2026-08-10

### Changed

- P0 security: `onAfterRender` skips non-HTML, JSON, XML, feed, and non-`text/html`
  responses
- P0 security: privacy and cookie policy URLs accept only `http(s)` or
  site-relative paths
- Developer API: `isCategoryEnabled`, `isGranted`, `whenGranted`, and
  `capabilities` flags

### Fixed

- Documented dynamic embed limitations and the public `DevArtConsent` API in
  `docs/limitations.md`

## 0.1.41 — 2026-08-10

### Added

- GitHub update server metadata (`update.xml`, `changelog.xml`, `LICENSE.txt`)
- Package update server URL for `joomla-devart-consent`

### Changed

- JED compliance: GPL headers and JEXEC guards on all PHP files
- Blocked HTML storage uses JSON attribute encoding instead of `base64_encode`
- Package, component, and plugin manifests declare Joomla 6 target platform

### Fixed

- CSV export no longer writes a UTF-8 BOM flagged by JAMSS
- Early HTML detection avoids literal iframe tag strings in plugin guards

## 0.1.40 — 2026-08-10

### Changed

- Tools and consents headers now use the packaged WebP assets
- Active DevArt Consent submenu items use `#3b963f`
- Dashboard shows only the essential status summary cards
- Administrator footer spacing and centering apply on every component view

## 0.1.39 — 2026-08-10

### Changed

- Administrator dashboard, tools, and consents views now use centered responsive
  submenu headers without a white background
- Administrator footer spacing and alignment match other DevArt extensions
- Active DevArt Consent submenu items use the standard DevArt orange highlight

## 0.1.38 — 2026-08-10

### Added

- Administrator dashboard status panel with frontend protection summary,
  blocking-layer count, package/engine version, and registry totals
- Per-area configuration status for core, categories, Google services,
  marketing pixels, external embeds, custom rules, appearance, and technical
  settings
- Dashboard warnings when the system plugin or privacy banner is disabled

## 0.1.37 — 2026-08-10

### Added

- Built-in frontend messages for English (`en-GB`) and Greek (`el-GR`) via
  `frontend-messages.json`
- Floating privacy-settings button icon selection (gear, cookie, shield, sliders)
- Theme-coloured floating button using the active DevArt primary colour

### Changed

- Removed the redundant Messages admin tab; frontend wording is now provided by
  built-in language packs only
- Updated administrator and QA documentation for milestones I–J

## 0.1.36 — 2026-08-10

### Added

- Custom blocking rules tab with up to 30 administrator-defined patterns for
  script URLs, inline scripts, and iframe sources
- Consent category mapping per custom rule (analytics, advertising, external)

## 0.1.35 — 2026-08-10

### Changed

- Clarified admin labels separating marketing pixels from external embeds

## 0.1.34 — 2026-08-10

### Added

- Marketing pixel auto-blocking for Meta Pixel, TikTok Pixel, and LinkedIn
  Insight Tag

## 0.1.33 — 2026-08-10

### Changed

- Simplified Google services to ON/OFF auto-detect blocking (no IDs, no managed
  injection)
- Added Google Ad Manager (GPT) support

## Earlier Baseline

- Migrated the repository baseline to the DevArt Consent source.
- Replaced inherited product metadata and documentation with Consent-specific
  repository information.
- Aligned package, component, and plugin PHP minimum metadata with PHP 8.3.
