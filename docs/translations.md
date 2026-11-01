# DevArt Consent — Translations

English (`en-GB`) is the source language for all extension strings.

## Supported Languages

| Language | Tag | Frontend banner | Administrator / installer | Status |
|----------|-----|-----------------|---------------------------|--------|
| English | `en-GB` | Yes | Yes | Source language |
| Greek | `el-GR` | Yes | Yes | Curated; protected from overwrite |
| French | `fr-FR` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| German | `de-DE` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Spanish | `es-ES` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Italian | `it-IT` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Brazilian Portuguese | `pt-BR` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Czech | `cs-CZ` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Dutch | `nl-NL` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Polish | `pl-PL` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Russian | `ru-RU` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Ukrainian | `uk-UA` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Japanese | `ja-JP` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Turkish | `tr-TR` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |
| Chinese (Simplified) | `zh-CN` | Yes | Yes | Machine-generated (Argos + Google fallback); review welcome |

## Generation Workflow

Machine-generated locales are produced from `en-GB` with:

1. `scripts/i18n/translate-ini.py` — Argos Translate first, Google GTX fallback
2. `scripts/i18n/translate-frontend-messages.py` — same pipeline for `media/data/frontend-messages.json`
3. Placeholder and technical-token protection (`%s`, `%d`, product names, etc.)
4. `scripts/i18n/validate-ini.php` — key parity and placeholder parity
5. `scripts/i18n/validate-frontend-messages.php` — frontend pack key parity

Use `python3 scripts/i18n/translate-ini.py --refresh-identical` to re-translate only
keys whose target value still matches English (keeps real translations). Technical
IP-source option labels (`REMOTE_ADDR`, `CF-Connecting-IP`) stay English by design.

Protected locales (`en-GB`, `el-GR`) are never overwritten by the generators.

## Notes

- Placeholders (`%s`, `%d`) must remain unchanged in every translation.
- Technical product names such as DevArt, Google, GTM, AdSense, YouTube and Joomla should stay intact.
- The words `cookie` / `Cookies` are protected as technical tokens so machine translation does not turn them into food words.
- Ukrainian (`uk-UA`) prefers Google GTX because the Argos Ukrainian package returns unreliable short-string output.
- Frontend banner strings live in `frontend-messages.json` and are selected by active Joomla language tag.
- Administrator and frontend translations for machine-generated locales are open to native-speaker review.
- Translation-only milestones must not alter consent runtime logic beyond language packaging and notes.
