# myStateSchedulerWeb_test — repo handbook

> Part of the My State Scheduler workspace. Read the workspace umbrella
> `CLAUDE.md` first; this file covers only this repo's internals.

## Purpose
This repo is the **staging** (test) copy of the My State Scheduler static
landing site. It mirrors `web/myStateSchedulerWeb/` (the production site) and
is used to validate changes before they go live at
`https://mystatescheduler.com/`. The staging repo has its own GitHub remote
(`techbyfloig/myStateSchedulerWeb_test`) and its own deployment target
(separate from production). It is not a secondary branch — it is a separate
repository that tracks the same content with environment-specific overrides in
`config.js`.

The production counterpart of this site lives in `web/myStateSchedulerWeb/`.
See the "Mirror relationship" note in the Conventions section.

## Tech stack
- **Languages**: HTML5, CSS3, vanilla JavaScript (ES6+)
- **Fonts**: Google Fonts — DM Sans, Fraunces (loaded via `<link>` with
  `preconnect` hints)
- **Contact form**: Formspree (`https://formspree.io/f/mreawawg`) — a
  separate staging endpoint, distinct from the production endpoint
- **Build tool**: None — plain static site; files are served directly
- **Analytics**: None — no Vercel analytics or Google Analytics scripts in
  staging
- **Runtime**: any modern browser

## Architecture
Static multi-page application (MPA). No framework, no bundler, no build step.

```
index.html       Landing page (home)
privacy.html     Privacy policy
terms.html       Terms of service
config.js        Shared runtime config object (loaded before page scripts)
```

Each HTML page is self-contained: styles are inline `<style>` blocks, scripts
are inline `<script>` blocks. `config.js` is the one shared external script and
must be loaded before any page script that calls `resolveConfig()`. All i18n
translations live inside each page's own `<script>` block.

**Known diff from production**: `404.html` does not exist in staging. Other
files absent from staging vs production: `sitemap.xml`, `robots.txt`, the
domain-verification token file, webp-format images (`image.webp`,
`image1.webp`, `image1-mobile.webp`, `image2.webp`), `favicon.png`,
`apple-touch-icon.png`, `og-image.png`. These are intentional omissions for
the staging environment.

## Directory / module structure
```
myStateSchedulerWeb_test/
├── index.html              # Marketing landing page
├── privacy.html            # Privacy policy (bilingual)
├── terms.html              # Terms of service (bilingual)
├── config.js               # Shared site config (staging values)
├── appicon.jpg             # App icon
├── image.png               # Hero images (PNG only; no webp variants in staging)
├── image1.png
└── image2.png
```

Ignore: `.git/`.

**Not present in staging** (present in production):
`404.html`, `sitemap.xml`, `robots.txt`, `favicon.png`, `apple-touch-icon.png`,
`og-image.png`, `c3d02a5b46d64eb5b841f8941940c19e.txt`, webp image variants,
`.idea/`.

## Build
No build step. There is no `package.json`, no bundler, and no compilation.
To "build" means verifying the static files are valid before deploying.

Prerequisites: none beyond a modern browser or an HTML validation tool. No
Node, JDK, or other runtime required.

## Run
No local server is strictly required — files can be opened directly in a
browser via `file://`. For a closer-to-production experience, serve the repo
root with any static file server, for example:

```bash
# Using Python (available on macOS by default)
python3 -m http.server 8080 --directory /path/to/myStateSchedulerWeb_test
# Then open http://localhost:8080
```

No environment variables or secrets are needed to run the site locally.
`config.js` contains only non-secret site configuration.

Staging URL: confirm in the deployment platform dashboard (separate from the
production Vercel project).

## Test
No automated test suite exists. "Green" for this repo means:

1. **HTML validation** — run each HTML file through the W3C Markup Validator
   (`https://validator.w3.org/nu/`) or a local validator such as
   `npx html-validate index.html privacy.html terms.html`. No errors
   permitted; warnings should be reviewed.

2. **Link check** — verify all internal and external links are reachable.
   Tool: `npx linkinator . --recurse` (run from the repo root with a local
   server active). Dead links are a defect.

3. **i18n completeness check** — manually toggle EN↔ES on each page and
   confirm every string switches. Any untranslated string (falls back to
   English key or blank) is a defect. See Localization section for keys.

4. **Visual smoke test** — open each page in a browser and verify layout at
   mobile (375 px) and desktop (1280 px) widths.

Note: because `404.html` is absent from staging, the 404-page path is not
testable here; validate it in production only.

No step requires any build or install. A task is not complete until all four
checks pass.

## Dependencies & integrations
**Internal:**
- This repo is the **staging mirror** of `web/myStateSchedulerWeb/`
  (production). Content changes to one must be propagated to the other, with
  the known permanent diffs documented in this handbook.
- No dependency on any server repo at build or serve time.

**External:**
- **Google Fonts** — DM Sans and Fraunces; loaded via `fonts.googleapis.com`
  with `preconnect` hints
- **Formspree** — staging contact-form endpoint `mreawawg`
  (`https://formspree.io/f/mreawawg`); intentionally different from
  production's `xbdqlvly`
- No Vercel analytics, no Google Analytics in staging
- No database, no API calls at runtime

## Conventions
- **Branch naming**: `feature/<spec-id>-<short-name>`; never commit directly
  to `main`.
- **Mirror relationship**: this repo (`myStateSchedulerWeb_test`) is the
  **staging** mirror of `myStateSchedulerWeb` (production). Content changes
  must flow between the two repos. The following are **permanent, intentional
  differences** and must never be "fixed" by copying production into staging:
  - `404.html` — production only
  - `sitemap.xml`, `robots.txt` — production only (SEO assets; staging should
    not be indexed)
  - `c3d02a5b46d64eb5b841f8941940c19e.txt` — production domain-verification
    token; do not add to staging
  - Webp image variants — not present in staging
  - `favicon.png`, `apple-touch-icon.png`, `og-image.png` — not present in
    staging
  - `config.js` Formspree endpoint (`mreawawg` vs `xbdqlvly`) — use the
    staging endpoint in this repo
  - `config.js` stats values — staging may carry different test values
  - Analytics scripts — production has Vercel + GA; staging has neither
- **Accessibility**: semantic HTML, ARIA labels on interactive controls,
  keyboard navigability. Regressions are defects.
- **Pass all four test checks before marking a task complete.**

## Localization (i18n)
This site is **user-facing** and fully bilingual (English and Spanish).
A missing locale is a defect — every displayed string must exist in both
languages.

**Mechanism** (same pattern on all pages, identical to production):

1. Every translatable DOM element carries a `data-i18n="<key>"` attribute.
   Its default text content is the English string, written directly in the
   HTML.
2. Elements whose translated value contains HTML (inline tags, SVG) also carry
   `data-i18n-html="true"`; those are swapped via `innerHTML` rather than
   `textContent`.
3. Input placeholders use `data-i18n-placeholder="<key>"`.
4. Each page contains an inline `const translations = { es: { "<key>": "<ES
   string>", … } }` JavaScript object with every Spanish string keyed by the
   same `data-i18n` keys.
5. `initTranslations()` runs at page load and snapshots the English strings
   from the live DOM into an `englishOriginals` map.
6. `setLanguage(lang)` iterates all `[data-i18n]` elements and applies either
   the Spanish translation or the English original.
7. `toggleLang()` reads `localStorage.getItem('lang')` and flips between `'en'`
   and `'es'`. The choice persists across sessions via `localStorage`.
8. The nav exposes an EN/ES pill toggle (`.lang-toggle`) that calls
   `toggleLang()`.

**Config-driven date strings**: `config.js` exposes `CONFIG.legal.effectiveDate`
(English) and `CONFIG.legal.effectiveDateEs` (Spanish). `{{key}}` template
placeholders in translation strings are resolved by `resolveConfig()`.

**Where translations live**: inside each `.html` file in the inline `<script>`
block, in the `translations.es` object. There is no separate translations file.
When adding or editing a string, update both the HTML default text (EN) and the
`translations.es` entry (ES) in the same file.

**Staging-specific note**: the staging `index.html` contains a `nav-trust`
navigation link that is absent from production's `index.html`. When syncing
changes between repos, verify that this nav difference is intentional and not
a merge artifact.

## Gotchas & secrets handling
**Gotchas:**
- `config.js` must be loaded before any page script that calls
  `resolveConfig()` or reads `CONFIG`. Loading order matters.
- `translations.es` lives inline in each HTML file, not in `config.js`. Adding
  a new page means writing a new translation block from scratch.
- Staging and production `config.js` intentionally differ in Formspree
  endpoint and stats values — do not overwrite staging's `config.js` with the
  production one without updating those fields.
- Staging does not have `404.html`. If a custom 404 is needed in staging, add
  it separately; do not copy blindly from production without reviewing the
  `favicon.png` and `apple-touch-icon.png` references that staging lacks.
- The staging `index.html` has a `nav-trust` nav link not present in
  production. Review this before syncing pages between repos.
- No analytics scripts exist in staging — do not add Vercel analytics or
  Google Analytics IDs here.

**Secret files present in this repo**: None. `config.js` contains only
non-sensitive configuration (company address, public API form endpoints,
placeholder store links). No `.env` files, no API keys, no credentials exist
in this repo.

**Rule**: agents must never read, write, or commit any secret. If a secret
ever appears in this repo by mistake, flag it to the stakeholder immediately
rather than committing or pushing.

## Ownership
Owner agent: **web-dev**
Human point of contact: Leonel Florin (florinleonel@techbyfloig.com),
TECHBYFLOIG LLC
