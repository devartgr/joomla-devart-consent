# DevArt Consent for Joomla

Professional GDPR consent management package for Joomla 6, designed for municipalities, organizations, publishers, agencies, business websites, and high-traffic production environments behind full-page cache or Cloudflare.

![Joomla](https://img.shields.io/badge/Joomla-6.x-blue)
![PHP](https://img.shields.io/badge/PHP-8.3%2B-green)
![Release](https://img.shields.io/badge/Version-1.1.2-orange)
![License](https://img.shields.io/badge/License-GPLv3-red)

---

## Overview

DevArt Consent is a modern Joomla 6 consent layer built for managing privacy banners, blocking third-party scripts and embeds, and applying visitor consent on production websites without breaking page cache.

The extension is designed for environments where performance, cache compatibility, GDPR correctness, administrator usability, and predictable updates are essential.

DevArt Consent supports:

- Cache-first consent banner and floating privacy button
- Server-side blocking for Google services, marketing pixels, and external embeds
- Scoped dynamic embed observer for JavaScript-created nodes
- Google Consent Mode v2
- Optional consent registry with CSV export
- Built-in frontend messages for 15 languages (en-GB, el-GR, and 13 additional locales)
- DevArt appearance themes
- Administrator dashboard hubs for Tools, Consents and Settings
- Automatic Joomla cache clear on policy version bump

The package includes a component and a system plugin.

---

## Version 1.1.2

DevArt Consent 1.1.2 is the recommended stable release for Joomla 6 production sites.

### Version 1.1.2 Highlights

- Reliable multilingual policy URL saves (single JSON language map + one Privacy/Cookies editor)
- Policies language selector restored after Apply/Save
- Fixes for Greek (and other non-default languages) not persisting, and flaky English display after save
- Safe update from 1.1.1; existing URL values are preserved

## Version 1.1.1

DevArt Consent 1.1.1 added per-language privacy and cookie policy URLs for multilingual sites.

### Version 1.1.1 Highlights

- Per-language privacy and cookie policy URLs (Forms-style language selector with flags)
- Banner policy links resolve from the active site language
- Fixes for language labels and Privacy URL persistence on multilingual sites
- Safe update from 1.1.0; legacy single URL fields remain as fallback

## Version 1.1.0

DevArt Consent 1.1.0 consolidated the post-1.0.1 milestone series for Joomla 6 production sites.

### Version 1.1.0 Highlights

- Administrator dashboard hubs for Tools, Consents and Settings with DevArt-coloured icons
- Non-hub Options button under the hubs
- Activated embeds no longer keep a grey placeholder frame; original iframe dimensions are preserved
- Built-in translations for fr-FR, de-DE, es-ES, it-IT, pt-BR, cs-CZ, nl-NL, pl-PL, ru-RU, uk-UA, ja-JP, tr-TR and zh-CN (plus en-GB and el-GR)
- Public consent registry endpoint hardened with FPC-safe token, same-site checks and Cache-Control no-store
- Visitor hashing uses `hash_hmac` with the Joomla secret as the HMAC key
- Safe update from `1.0.1` and earlier `1.0.x` packages

---

## Package Contents

The installable package includes:

- `com_devartconsent` — Administrator component (dashboard, settings, tools, consents)
- `plg_system_devartconsent` — System plugin for banner injection, blocking pipeline, and frontend runtime

Always install or update using the full `pkg_devartconsent` package.

Do not install the standalone component ZIP on a clean website.

---

## Consent Banner

The frontend banner supports:

- Accept all
- Reject all
- Custom preferences
- Consent revocation through the floating privacy button
- Privacy policy and cookie policy links
- Built-in messages for `en-GB`, `el-GR`, and 13 additional locales
- DevArt appearance themes and radius presets
- Floating button icon selection (gear, cookie, shield, sliders)
- Banner positions: bottom, top, and centered modal

Consent is stored in a first-party browser cookie. The server does not require Joomla sessions, per-visitor database writes for banner display, or `Vary: Cookie`.

---

## Cache-First Architecture

DevArt Consent is designed for full-page cache, reverse proxies, and Cloudflare.

Key characteristics:

- The same HTML is served to every anonymous visitor
- Consent is applied client-side in the browser
- No `Vary: Cookie` requirement
- No per-request consent database writes for banner display
- Cached HTML remains safe for visitors who have not yet granted consent

This makes DevArt Consent suitable for high-traffic Joomla websites where traditional session-based consent tools break page cache.

When the policy version changes, Joomla page cache groups are cleared automatically on save. CDN or reverse-proxy cache purge remains manual.

---

## Server-Side Blocking

DevArt Consent blocks protected third-party content in the final HTML rendered by Joomla before the response leaves PHP.

Blocking layers run in this order:

1. Google services
2. Marketing pixels
3. External embeds
4. Custom administrator rules

Blocked markup is sent as cache-safe placeholders or `type="text/plain"` scripts and is restored in the browser only after the relevant consent category is granted.

### Google Services

Per-service ON/OFF auto-detect blocking for:

- Google Analytics (GA4 and legacy patterns)
- Google Tag Manager
- Google Ads
- Google AdSense
- Google Ad Manager (GPT)

### Marketing Pixels

Auto-detect blocking for:

- Meta Pixel
- TikTok Pixel
- LinkedIn Insight Tag

### External Embeds

Configurable provider blocking for:

- YouTube
- Vimeo
- Google Maps
- Facebook
- Instagram
- TikTok
- X / Twitter
- OpenStreetMap
- SoundCloud
- Spotify

### Custom Blocking Rules

Administrators can define up to 30 custom patterns for:

- Script URLs (`src`)
- Inline scripts
- Iframe sources

Each rule maps to analytics, advertising, or external consent categories.

---

## Dynamic Embed Observer

JavaScript-created `script` and `iframe` nodes are intercepted by a scoped runtime observer when blocking layers are enabled.

Observer behaviour:

- Direct `SCRIPT` and `IFRAME` nodes are inspected immediately
- Container subtrees use a debounced scan (150 ms)
- The observer is exposed through `DevArtConsent.capabilities.dynamicEmbedObserver`

Dynamic blocking is best-effort client-side protection. A browser may start a network request before the observer rewrites a dynamically inserted `src`. For custom widgets, use `window.DevArtConsent.whenGranted()`.

See `docs/limitations.md` for full runtime limits.

---

## Google Consent Mode v2

When enabled, DevArt Consent:

- Sets default denied consent states before other head scripts run
- Initialises `dataLayer` and `gtag` once
- Supports `wait_for_update`
- Pushes consent updates before blocked Google scripts are restored

This integrates with Google Analytics, Google Ads, and other Google tags blocked until consent is granted.

---

## Consent Registry

Optional server-side consent recording is available through component settings.

Registry features include:

- `com_ajax` recording endpoint with FPC-safe registry token and same-site checks
- Hashed visitor key identification via `hash_hmac`
- CSV export from the administrator consents view
- Filters by decision, source, and date
- No Joomla session required on the frontend
- Cache-Control no-store on registry AJAX responses

Registry recording is disabled by default.

---

## Administrator Area

The component administrator area includes:

- **Dashboard** — hubs for Tools, Consents and Settings, Options shortcut, and status info cards
- **Settings** — banner, categories, policies, appearance, Google services, marketing pixels, external embeds, custom rules, Google Consent Mode, consent registry
- **Tools** — operational utilities
- **Consents** — registry records and CSV export when enabled

Active DevArt Consent submenu items use the standard DevArt theme colours.

---

## Frontend Developer API

`window.DevArtConsent` is available after the deferred runtime script loads.

Useful methods include:

- `has(category)` / `isGranted(category)`
- `isCategoryEnabled(category)`
- `whenGranted(category, callback)`
- `getState()`
- `acceptAll()`, `rejectAll()`, `save(categories)`, `revoke()`
- `on(event, callback)`
- `capabilities.staticHtmlBlocking`
- `capabilities.dynamicEmbedObserver`

See `docs/limitations.md` for examples and integration guidance.

---

## Frontend Performance

DevArt Consent is designed for large Joomla websites and high-traffic environments.

Performance characteristics include:

- Per-request runtime config cache
- Cheap pre-checks before expensive regex passes
- Per-layer skip when markup cannot match
- In-request JSON catalogue cache
- Lightweight native JavaScript IIFE
- No jQuery dependency
- Namespaced CSS
- Cloudflare-friendly same-HTML rendering
- Joomla page cache compatibility

---

## Joomla-Native Architecture

DevArt Consent follows modern Joomla development patterns, including:

- Joomla 6 MVC architecture
- Namespaced PHP classes with strict types
- Joomla service provider architecture
- Joomla Web Asset Manager where appropriate
- Joomla Form API
- Joomla ACL
- Joomla database APIs
- Joomla administrator Atum layouts
- CSRF protection
- Proper input filtering
- Escaped output
- `DatabaseInterface` dependency injection

No Joomla 3, 4, or 5 compatibility layer is included.

---

## Security

Security measures include:

- `PolicyUrlHelper` rejecting `javascript:`, `data:`, and parent-directory traversal in policy URLs
- `RenderedBodyGuard` excluding non-HTML, JSON, XML, and feed responses from banner injection
- ACL enforcement for administrator access
- CSRF protection for administrator actions and FPC-safe registry request guards
- Malformed and expired consent cookie invalidation on the client
- Controller to service to storage flow for consent registry
- CSP nonce synchronisation through `CspNonceHelper` when Joomla nonce generation is enabled
- Installer checksum support through update metadata SHA-256

JAMSS may report one warning for iframe-matching PCRE used to block third-party embeds. This is expected core behaviour for a consent extension, not obfuscated or malicious code.

---

## Requirements

- Joomla 6.0 or newer
- PHP 8.3 or newer
- MySQL or MariaDB supported by Joomla 6
- A modern browser for administrator and frontend interfaces

Optional:

- Joomla **System - HTTP Headers** plugin with CSP nonce generation enabled
- CDN or reverse proxy in front of the site (supported; manual purge after policy version changes)

---

## Installation

1. Download the latest package:

   `pkg_devartconsent_v1.1.2.zip`

2. Open the Joomla administrator.

3. Go to:

   `System → Install → Extensions`

4. Upload and install the **full package**.

5. Confirm that **plg_system_devartconsent** is enabled.

6. Open:

   `Components → DevArt Consent`

7. Configure the banner and blocking layers in **Settings**.

8. Test the frontend in a private browser window before going live.

The package supports installation and updates through the standard Joomla Extensions installer.

---

## Updating

DevArt Consent uses the standard Joomla update system.

Update server:

`https://raw.githubusercontent.com/devartgr/joomla-devart-consent/main/update.xml`

Before updating a production website:

- Create a complete backup
- Test the update on a staging environment
- Verify the administrator dashboard and settings
- Verify banner accept, reject, preferences, and revoke on the frontend
- Verify Google, marketing, and external blocking before and after consent
- Purge CDN or reverse-proxy cache after policy version changes
- Complete `qa/csp-verification.md` if CSP nonces are enabled

Version 1.1.2 is a safe update from `1.1.1` and `1.1.0` and earlier `1.0.x` packages.

---

## Download

Latest release:

`pkg_devartconsent_v1.1.2.zip`

GitHub releases:

https://github.com/devartgr/joomla-devart-consent/releases

Direct download:

https://github.com/devartgr/joomla-devart-consent/releases/download/v1.1.2/pkg_devartconsent_v1.1.2.zip

SHA-256:

`04501080c39b40449fc6a6553342520afdbab1660f853cd8c0907f154b2eb638`

---

## Support and Documentation

Project repository:

https://github.com/devartgr/joomla-devart-consent

Additional documentation in the repository:

- `docs/translations.md`
- `docs/stable-release.md`
- `docs/limitations.md`
- `docs/architecture.md`
- `qa/stable-signoff.md`
- `qa/csp-verification.md`
- `qa/runtime-matrix.md`

Website:

https://devart.gr

Before reporting an issue, include:

- Joomla version
- PHP version
- Database type and version
- DevArt Consent version
- Cache or CDN stack in use (Joomla cache, Cloudflare, other)
- Relevant error message or browser console output
- Steps required to reproduce the issue

Do not include passwords, private keys, access tokens, or other sensitive information.

---

## License

DevArt Consent is released under the GNU General Public License version 3 or later.

See the included license file for complete licensing information.

---

## Author

**Kostas Stathopoulos — DevArt**

Website: https://devart.gr
