# Jekyll Website Template

Headless-ish personal-site base: Jekyll + [Decap CMS](https://decapcms.org/)
admin, swappable themes, bilingual (EN/PT-BR) out of the box. Use as a
GitHub template repo — one copy per site/person.

Design decisions and rationale: [PLAN.md](PLAN.md). Theme swap rules:
[THEME-CONTRACT.md](THEME-CONTRACT.md). CMS auth setup:
[OAUTH.md](OAUTH.md).

## What's in the base

- Jekyll site w/ `minima` as a reference theme (swap via `_config.yml`).
- Content model: `_posts` (blog), `_projects` (portfolio), `pages/`
  (About/Contact/...), all bilingual (`slug.en.md` / `slug.pt-br.md`).
- `_data/navigation.yml`, `_data/social.yml`, `_data/settings.yml` —
  CMS-editable site chrome/copy.
- `admin/` — Decap CMS, GitHub backend, direct-commit publishing (no PR
  drafts — simplest for non-technical editors).
- `.github/workflows/` — CI build check + deploy workflows for GitHub
  Pages and Netlify (delete whichever you don't use).

## New site from this template — setup checklist

1. **Use this template** on GitHub → new repo under your own account.
2. **Local dev**: `bundle install`, then `bundle exec jekyll serve`.
3. **Edit `_config.yml`**: `title`, `description`, `url`, and (optional)
   `analytics.ga4_id`.
4. **Pick a theme**: leave `theme: minima`, or point `remote_theme:` at
   another Jekyll theme gem. Check it against
   [THEME-CONTRACT.md](THEME-CONTRACT.md) — following it keeps content
   and CMS config untouched; not following it may need the override
   files described there adapted to the new theme.
5. **Wire the CMS**: edit `admin/config.yml`'s `backend.repo` to this
   site's own `owner/repo`. Auth setup: [OAUTH.md](OAUTH.md) — reuses
   the shared `sveltia-cms-auth` worker, one-time maintainer setup.
6. **Pick a hosting target**:
   - **GitHub Pages**: repo Settings → Pages → Source = "GitHub
     Actions". Keep `deploy-ghpages.yml`, delete `deploy-netlify.yml`.
   - **Netlify**: create the Netlify site, add repo secrets
     `NETLIFY_AUTH_TOKEN` / `NETLIFY_SITE_ID`. Keep
     `deploy-netlify.yml`, delete `deploy-ghpages.yml`.
   - **Private server**: not yet templated (see PLAN.md open items) —
     `ci.yml` still validates builds; add your own deploy step once a
     target is chosen.
7. **Push to `main`** — deploy workflow runs automatically.
8. Log into `/admin/` on the deployed site to confirm CMS auth works
   end-to-end, then replace the sample post/project/pages content.

## Local development

```bash
bundle install
bundle exec jekyll serve
```

Site at `http://localhost:4000`. `/admin/` won't authenticate locally
against the production OAuth worker unless it's added to that worker's
`ALLOWED_DOMAINS` — editing is normally done against the deployed site.

## Updating a site from this template later

No automated sync in v1. Add the base as a remote and merge/cherry-pick:

```bash
git remote add upstream https://github.com/MartinSchoeler/jekyll-website-template.git
git fetch upstream
git merge upstream/main   # resolve conflicts against your own content
```

## Known open items

See "Open items — still outstanding" in [PLAN.md](PLAN.md) (GA4 consent
banner, private-hosting deploy mechanics) and "Known limitations (v1)" in
[THEME-CONTRACT.md](THEME-CONTRACT.md).
