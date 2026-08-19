# Theme Contract

This base is "headless": content, CMS config, i18n, and deploy plumbing are
theme-independent. A theme that honors this contract can be swapped in via
`_config.yml` (`theme:`/`remote_theme:`) without touching content or
`admin/config.yml`.

## What the base guarantees (theme can rely on)

### Layouts requested by content
- `page` — used by everything in `pages/`.
- `post` — used by everything in `_posts/`.
- `project` — custom collection layout, defined by base itself
  (`_layouts/project.html`), extends `default`.
- `home` — defined by base itself (`_layouts/home.html`), extends `default`.

A theme must supply (or inherit) a `default` layout that all of the above
extend — this is standard Jekyll theme-gem convention.

### Front matter fields
- **Posts** (`_posts/*.md`): `title`, `lang`, `date` (from filename).
- **Projects** (`_projects/*.md`): `title`, `lang`, `summary` (optional),
  `link` (optional, external URL).
- **Pages** (`pages/*.md`): `title`, `lang`, `permalink`.

All three are also split one-file-per-locale: `slug.en.md`, `slug.pt-br.md`
— see [i18n](#i18n) below.

### Data files (`_data/`) — CMS-editable, theme-consumable
- `navigation.yml` — object keyed by language code, each a list of
  `{title, url}`.
- `social.yml` — `{links: [{name, url, icon}]}`.
- `settings.yml` — `{hero: {<lang>: {title, subtitle}}, footer: {<lang>:
  {text}}}`.

### Includes theme can call
- `nav.html` — renders `site.data.navigation[lang]`.
- `social-links.html` — renders `site.data.social.links`.
- `lang-switcher.html` — MVP language switcher (links to each language's
  homepage, not per-page — see the include's own comment).
- `hreflang.html` — `<link rel="alternate">` tags for the languages in
  `site.languages`.
- `ga4.html` — no-ops unless `site.analytics.ga4_id` is set.

### CSS custom properties (`assets/css/base.css`)
`--color-bg`, `--color-text`, `--color-muted`, `--color-accent`,
`--color-border`, `--font-body`, `--font-heading`, `--font-mono`,
`--space-1`…`--space-6`, `--content-max-width`, `--radius`.

Base only resets the box model and declares these tokens — it renders no
visual opinion of its own. Themes should read/override these variables
rather than override with higher-specificity/`!important` rules.

### i18n
- Plugin: `jekyll-polyglot`. `site.languages`, `site.default_lang` set in
  `_config.yml`.
- `site.posts`, `site.projects`, etc. contain **all languages mixed** —
  Polyglot tags each item's `.lang` but does not split the arrays. Any
  listing template must filter, e.g. `site.posts | where: "lang", lang`.
- Non-default-lang URLs are prefixed (`/pt-br/...`); default lang is not.
- **Polyglot auto-rewrites root-relative links to the current page's
  language** ("automatic URL relativization") — it post-processes the
  *rendered HTML* and prepends the active lang to every `href="/..."`,
  so template authors don't have to localize every link by hand. This
  is right for normal content links but wrong for anything that must
  deliberately point at a DIFFERENT language (the language switcher,
  hreflang tags): a leading space right after the opening quote
  (`href=" /..."`) defeats Polyglot's rewrite regex while staying a
  normal working link (browsers trim leading href whitespace) — see
  `lang-switcher.html`/`hreflang.html` and [Polyglot's docs on
  it](https://github.com/untra/polyglot#preventing-automatic-relativization).

## What a new theme must provide

- A `default` layout (header/nav/footer chrome) that calls `{% include
  nav.html %}`, `{% include lang-switcher.html %}`, `{% include
  social-links.html %}`, and injects `{% include hreflang.html %}` /
  `{% include ga4.html %}` in `<head>`.
- Its own visual CSS, layered on top of `assets/css/base.css`'s custom
  properties (link `base.css` first, theme stylesheet after).
- Layouts named `default`/`page`/`post` at minimum (base supplies
  `home`/`project` itself, extending the theme's `default`).

## Reference integration: minima

The base ships wired to [minima](https://github.com/jekyll/minima) as a
working proof that the contract above is sufficient. Because minima
doesn't know about this base's data files, three of minima's own include
files are **overridden** by same-named files in this repo (Jekyll loads
project files over gem files with identical paths):

- `_includes/head.html` — adds `base.css`, `hreflang.html`, `ga4.html`.
- `_includes/header.html` — calls `nav.html` / `lang-switcher.html`
  instead of minima's own nav logic.
- `_includes/footer.html` — calls `social-links.html` / renders
  `settings.yml` footer text instead of minima's own footer config.
- `_layouts/home.html` — renders `settings.yml` hero + a locale-filtered
  post list, instead of minima's own home layout.

**Swapping themes**: if the new theme already calls the includes above on
its own, delete these four override files. If not, adapt them to the new
theme's include/layout names — this is real, expected work per theme
swap, not a fully zero-touch operation. This is the honest cost of
"headless + a real theme's private markup" — flagged here rather than
hidden.

## Known limitations (v1)

- `lang-switcher.html`/`hreflang.html` link to each language's homepage,
  not the current page's translation. A precise version needs Polyglot's
  `page.ref` mechanism.
- Decap's i18n `multiple_files` structure doesn't template a `lang:`
  front-matter default per locale automatically — editors should confirm
  the saved file's `lang:` value matches the locale tab they edited.
- **Verified/fixed**: Jekyll only strips the `.md` extension when
  deriving a slug, so `welcome.pt-br.md` (Decap's i18n `multiple_files`
  naming) would otherwise leak `.pt-br` into the URL, and Polyglot's
  automatic non-default-lang URL prefixing double-stacks with any
  manually lang-prefixed `permalink:`. Fix applied: posts/projects carry
  an explicit `slug:` front-matter field (also exposed as a CMS field,
  `i18n: duplicate`, so it's set once and copied across locale files),
  and localized pages/index files use the *unprefixed* path in
  `permalink:` — Polyglot adds the `/pt-br/` prefix itself.
