# Personal Jekyll Website Base — Plan

Status: planning only, no code yet. Decisions below locked from Q&A; open items flagged at bottom.

## 1. Goals

- One base repo, used as **GitHub template repo**. You + friends each get own copy (own repo, own Decap login, own deploy target).
- Base = content/CMS/plumbing only. Zero hardcoded visual styling. Look/feel comes from swappable **Jekyll themes** (remote_theme/gem convention).
- Must work unmodified across 3 hosting targets: GitHub Pages, Netlify, private server.
- Decap CMS as admin UI, git-backed (no DB).

## 2. Repo model

- Base repo marked as GitHub "Template repository".
- New site = "Use this template" → new repo → run setup checklist (see §9).
- No shared runtime dependency between sites. Updates from base flow via `git remote add upstream` + manual merge/cherry-pick (documented in README, not automated in v1).

## 3. Directory structure (target)

```
/
├─ _config.yml            # site config incl. remote_theme, plugins, i18n, analytics toggle
├─ _data/
│  ├─ navigation.yml       # nav links (CMS-editable)
│  ├─ social.yml           # social links (CMS-editable)
│  └─ settings.yml         # site-wide settings (hero text, footer, etc.)
├─ _posts/                 # blog collection
├─ _projects/              # portfolio/projects collection (custom collection)
├─ pages/ (or root .md)    # static pages: about, contact, etc.
├─ assets/
│  ├─ css/
│  │  └─ base.css          # ONLY resets + CSS custom properties, no visual opinions
│  └─ uploads/              # Decap media folder, in-repo
├─ admin/
│  ├─ index.html            # Decap CMS entry
│  └─ config.yml            # Decap collections config
├─ i18n/ or _i18n/          # polyglot translation data
├─ .github/workflows/
│  ├─ build-deploy-ghpages.yml
│  ├─ build-deploy-netlify.yml (or rely on Netlify native build)
│  └─ build-only.yml        # artifact for private hosting
└─ PLAN.md / README.md
```

Themes are NOT vendored in this repo — pulled via `remote_theme:` (Pages-compatible) or `gem` theme in Gemfile, site owner picks in `_config.yml`.

## 4. Theme system — headless contract

- Base follows standard Jekyll theme convention: layouts/includes/sass are all **overridable** by whatever theme is set, base provides only fallback/minimal versions so site works theme-less too.
- Base defines a fixed set of **front-matter/data contracts** themes must render against (documented in `THEME-CONTRACT.md`, e.g. what fields a post/page/project has, what `_data/navigation.yml` shape looks like). Any theme following the contract is swappable without touching content or CMS config.
- Since we use GitHub Actions build (not native GH Pages build), we are NOT limited to the GH Pages plugin whitelist — full `remote_theme` gem ecosystem usable everywhere, including GH Pages target.

## 5. Styling architecture

- Base ships `assets/css/base.css`: CSS reset + CSS **custom properties** only (`--color-*`, `--space-*`, `--font-*`), no component/visual styling.
- Themes bring their own stylesheet, expected to consume/override those custom properties. Base never sets `!important` or fights specificity.
- No inline styles anywhere in base templates (matches your requirement).

## 6. Content model / collections

- `_posts` — blog.
- `_projects` — portfolio collection, output: true, own permalink pattern.
- Static pages — About/Contact/etc as top-level `.md` with front matter, editable via Decap "pages" collection (folder or file collection type per page).
- `_data/navigation.yml`, `_data/social.yml`, `_data/settings.yml` — CMS-editable via Decap "data" file collections.

## 7. i18n

- Plugin: **Polyglot** (`untra/jekyll-polyglot`) — active, works with collections, no GH Pages whitelist issue since we self-build via Actions.
- v1 ships **fully bilingual default content**: **EN + PT-BR** (confirmed) as working example across posts/pages/projects/data.
- Language switcher = base include, themes may restyle but not required to implement logic.
- Decap config: language-aware collections (Polyglot's per-locale front matter/file convention) so translated fields editable in CMS.

## 8. Decap CMS setup

- `admin/config.yml`: backend `github`, repo per-site (owner/repo, branch `main`), `base_url` pointing at the shared OAuth provider (§9).
- Media: `media_folder: assets/uploads`, `public_folder: /assets/uploads` — in-repo, no external service.
- Collections mirror §6: posts, projects, pages, and data-file collections for nav/social/settings.
- Editorial workflow: **direct-to-branch commits** (confirmed) — no PR/draft queue. Right call given non-technical friends: Decap already hides git, direct-commit means zero branch/merge concepts exposed to them.

## 9. Auth — shared OAuth provider

- One shared **OAuth proxy** you deploy once and maintain: **`sveltia-cms-auth`** (Cloudflare Worker, free tier, confirmed choice — supports Decap's `github` backend).
- Each site owner (you/friends) registers own GitHub OAuth App (or shares one) pointing callback at your worker URL; site's `admin/config.yml` `base_url` points at same worker regardless of where the site itself is hosted.
- This decouples auth from hosting target — works identically for GH Pages, Netlify, private server sites.
- Risk: single point of failure/maintenance is on you — noted in Open Items.

## 10. Deploy — per hosting target, all via GitHub Actions

- **GitHub Pages**: Actions workflow builds Jekyll (unrestricted plugins) → deploys via `actions/deploy-pages` (Pages source = "GitHub Actions", not legacy branch build).
- **Netlify**: Actions-driven deploy (confirmed) — build job → deploy step using Netlify CLI/action with site's `NETLIFY_AUTH_TOKEN`/`NETLIFY_SITE_ID` secrets. Consistent w/ other targets, no reliance on Netlify's native build.
- **Private hosting**: deferred (open item #4, unresolved by design — no private host committed yet). GH Pages ships first; private-host workflow (rsync/scp/ssh/Docker/whatever) gets designed once a real target exists. Base's build job stays host-agnostic (`_site` as artifact) so slotting in a deploy step later is additive, not a rework.
- Single shared "build" job/step reused across all 3 workflows (matrix or reusable workflow) to avoid drift.

## 11. Analytics

- **Google Analytics (GA4)**.
- `_config.yml`: `analytics.ga4_id:` — empty by default. Base include only injects GA script tag if id present, so sites without analytics stay clean.
- No cookie-consent banner built into base v1 (flag as open item — GA4 has consent implications).

## 12. New-site setup flow (documented in README, not automated)

1. "Use this template" → new repo.
2. Edit `_config.yml`: site title/url/lang/theme/ga4_id.
3. Point `admin/config.yml` repo field at new repo.
4. Register/point GitHub OAuth App at shared worker (or reuse existing app if same GitHub org).
5. Pick hosting target → enable matching workflow, fill host-specific secrets.
6. Pick a Jekyll theme, set `remote_theme:` (or Gemfile gem), confirm it satisfies `THEME-CONTRACT.md`.

## 13. CI checks

- Build validation (`jekyll build` must succeed) on every PR/push.
- Optional: `html-proofer` for broken links/images (recommend on, low cost).

## Open items — resolved

1. ~~Editorial workflow~~ → direct-commit (confirmed, §8).
2. ~~Second language~~ → EN + PT-BR (confirmed, §7).
3. ~~OAuth provider~~ → `sveltia-cms-auth` (confirmed, §9).
4. ~~Netlify path~~ → Actions-driven deploy (confirmed, §10).

## Open items — still outstanding

1. **Private-hosting deploy mechanics**: intentionally deferred — no private host committed yet. GH Pages ships first (§10); revisit once a real target/provider is picked.
2. **GA4 + consent**: add a minimal cookie-consent gate, or leave to each site owner? Default proposed: leave out of base v1, note in README.
3. **Theme contract document**: needs drafting (`THEME-CONTRACT.md`) before real theme-swap testing — propose as first build task.

## Suggested build order (once plan approved)

1. Core Jekyll skeleton + `_config.yml` + collections + data files.
2. `THEME-CONTRACT.md` + one reference theme wired via `remote_theme` to prove headless split.
3. Decap `admin/config.yml` (direct-commit workflow) + shared `sveltia-cms-auth` worker deployed once.
4. GitHub Actions build + GH Pages deploy (primary target, your own site — first real deployment).
5. Netlify Actions-driven deploy workflow.
6. Polyglot i18n wiring + EN/PT-BR demo content.
7. GA4 include + config toggle.
8. README/setup docs for friends adopting the template (non-technical-friendly, direct-commit CMS flow).
9. Private-host deploy workflow — once a concrete target is chosen (open item #1).
