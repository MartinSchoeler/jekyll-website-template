# Decap CMS Auth — shared `sveltia-cms-auth` worker

Decap's `github` backend needs an OAuth proxy regardless of where the
site itself is hosted (GH Pages, Netlify, private server all use the
same one). One shared instance, deployed once, reused by every site
built from this template.

## One-time setup (you, as maintainer)

1. **Create a GitHub OAuth App**: GitHub → Settings → Developer settings
   → OAuth Apps → New OAuth App.
   - Homepage URL: anything (e.g. the worker's URL, filled in after step 2).
   - Authorization callback URL: `https://YOUR-WORKER.workers.dev/callback`.
   - Note the Client ID and Client Secret.

2. **Deploy `sveltia-cms-auth`** to Cloudflare Workers:
   - Repo: https://github.com/sveltia/sveltia-cms-auth
   - `wrangler deploy` (or the repo's documented deploy button/flow).
   - Set secrets: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` (from step 1).
   - Optional: `ALLOWED_DOMAINS` to restrict which site origins may use
     this worker (recommended once multiple friends' sites share it).

3. Note the deployed worker URL — this is the `base_url` every site's
   `admin/config.yml` points at.

## Per-site setup (each new site from the template)

1. In `admin/config.yml`, set:
   ```yaml
   backend:
     name: github
     repo: <owner>/<repo>       # this site's own repo
     branch: main
     base_url: https://YOUR-WORKER.workers.dev
     auth_endpoint: auth
   ```
2. If `ALLOWED_DOMAINS` is set on the worker, add this site's deployed
   origin (GH Pages URL / Netlify URL / custom domain) to that list.
3. Confirm the GitHub account editing content has push access to the
   site's repo (repo collaborator, or org member).

## Must the worker be publicly reachable?

Yes — it must NOT sit behind Cloudflare Access/Zero Trust or any other
auth wall. Decap runs in the visitor's browser and calls the worker's
`/auth` and `/callback` endpoints directly; blocking that at the network
level breaks login before GitHub OAuth even starts.

This is safe because the worker's URL isn't the security boundary:

- **GitHub OAuth + repo permissions decide who can actually write.**
  Anyone can hit the worker and complete GitHub login, but committing
  still requires real collaborator/write access to the target repo —
  the worker never grants that itself.
- **`GITHUB_CLIENT_SECRET` stays server-side**, in the worker's env only
  — never sent to the browser. That's the sensitive value, not the URL.
- **`ALLOWED_DOMAINS`** (optional) restricts which site origins may call
  the worker — cuts down noise/abuse, not a hard security boundary.

Worst case with a fully public worker and no `ALLOWED_DOMAINS`: strangers
can run the OAuth flow and get their own GitHub token — harmless, no
path to writing to your repo without real GitHub write access there.

## Why this shape

Decoupling auth from hosting means the same CMS setup instructions apply
whether a friend's site ends up on GitHub Pages, Netlify, or a private
server — only the *deploy* workflow (`.github/workflows/`) differs per
host, not the *editing* setup. See [PLAN.md](PLAN.md) §9.

## Maintenance note

This shared worker is a single point of failure/maintenance, owned by
you (confirmed tradeoff — see PLAN.md open items). If it goes down, no
site's CMS can authenticate (published sites keep serving fine — this
only affects editing).
