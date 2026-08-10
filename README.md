# DevArt Consent for Joomla

Professional GDPR consent management package for Joomla 6, designed for municipalities, organizations, publishers, agencies, business websites, and high-traffic production environments behind full-page cache or Cloudflare.

![Joomla](https://img.shields.io/badge/Joomla-6.x-blue)
![PHP](https://img.shields.io/badge/PHP-8.3%2B-green)
![Release](https://img.shields.io/badge/Version-1.0.1-orange)
![License](https://img.shields.io/badge/License-GPLv3-red)

---

## Overview

DevArt Consent is a modern Joomla 6 consent layer for privacy banners, third-party script blocking, and visitor consent — without breaking page cache.

It is built for production sites where performance, Cloudflare compatibility, GDPR correctness, and simple administration matter.

**Main features:**

- Cache-first consent banner with accept, reject, preferences, and revoke
- Server-side blocking for Google services, marketing pixels, and external embeds
- Dynamic embed observer for JavaScript-created scripts and iframes
- Google Consent Mode v2
- Optional consent registry with CSV export
- Built-in English and Greek frontend messages
- DevArt appearance themes
- Automatic Joomla cache clear when policy version increases

The package includes a **component** and a **system plugin**.

---

## Version 1.0.1

Recommended stable release for Joomla 6 production sites.

**Highlights:**

- Faster dynamic blocking for directly inserted scripts and iframes
- Safer server-side blocking around template and SVG markup
- Improved CSP nonce compatibility with Joomla HTTP Headers plugin
- Frontend engine 2.0.1
- Safe update from 1.0.0, 1.0.0-rc.1, and 1.0.0-beta.*

---

## Package Contents

- `com_devartconsent` — Administrator area (dashboard, settings, tools, consents)
- `plg_system_devartconsent` — Frontend banner, blocking, and consent runtime

Always install or update the **full package** `pkg_devartconsent`.

Do not install the standalone component ZIP on a clean site.

---

## Consent Banner

- Accept all / Reject all / Custom preferences
- Floating privacy button with revoke
- Privacy and cookie policy links
- English and Greek built-in messages
- DevArt themes, radius presets, and icon options
- Positions: bottom, top, centered modal

Consent is stored in a first-party cookie. No Joomla session, no `Vary: Cookie`, no per-visitor database writes for banner display.

---

## Cache-First Design

- Same HTML for all visitors
- Consent applied in the browser
- Compatible with Joomla page cache, reverse proxies, and Cloudflare
- Safe cached HTML for visitors who have not consented yet

When you increase the **policy version**, Joomla page cache is cleared automatically. Purge CDN or reverse-proxy cache manually if you use one.

---

## Blocking

### Google services
Google Analytics, Google Tag Manager, Google Ads, Google AdSense, Google Ad Manager — each can be enabled or disabled separately.

### Marketing pixels
Meta Pixel, TikTok Pixel, LinkedIn Insight Tag.

### External embeds
YouTube, Vimeo, Google Maps, Facebook, Instagram, TikTok, X/Twitter, OpenStreetMap, SoundCloud, Spotify.

### Custom rules
Up to 30 administrator-defined patterns for script URLs, inline scripts, and iframe sources.

Blocked content is restored in the browser only after the relevant consent category is granted.

---

## Dynamic Embeds

JavaScript-created scripts and iframes are intercepted when blocking is enabled.

- Direct nodes are checked immediately
- Container subtrees use a short debounced scan

**Important:** dynamic blocking is best-effort. For custom widgets, use:

```javascript
window.DevArtConsent.whenGranted('external', () => {
  // load your widget here
});
```

---

## Google Consent Mode v2

When enabled, default denied states are set early, then updated after the visitor makes a choice — before blocked Google scripts are restored.

---

## Consent Registry (optional)

Record consent decisions server-side, export to CSV, filter by date and decision type. Disabled by default.

---

## Requirements

- Joomla 6.0+
- PHP 8.3+
- MySQL or MariaDB (as supported by Joomla 6)

---

## Installation

1. Download `pkg_devartconsent_v1.0.1.zip`
2. **System → Install → Extensions**
3. Install the full package
4. Enable **plg_system_devartconsent**
5. Open **Components → DevArt Consent → Settings**
6. Test the frontend in a private browser window

---

## Updating

Update server:

`https://raw.githubusercontent.com/devartgr/joomla-devart-consent/main/update.xml`

Before updating production:

- Back up the site
- Test on staging
- Verify banner and blocking on the frontend
- Purge CDN cache after policy version changes

---

## Download

**Latest:** `pkg_devartconsent_v1.0.1.zip`

**Releases:** https://github.com/devartgr/joomla-devart-consent/releases

**Direct:** https://github.com/devartgr/joomla-devart-consent/releases/download/v1.0.1/pkg_devartconsent_v1.0.1.zip

**SHA-256:** `f81f931e3d456b15de6fd16307ec27beed6e8bcc4d057c4d2ba339b19e155890`

---

## Support

**Repository:** https://github.com/devartgr/joomla-devart-consent  
**Website:** https://devart.gr

When reporting an issue, include Joomla version, PHP version, DevArt Consent version, cache/CDN stack, and steps to reproduce.

---

## License

GNU General Public License version 3 or later.

---

## Author

**Kostas Stathopoulos — DevArt**  
https://devart.gr
