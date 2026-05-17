# hsyvy — static site

Plain HTML/CSS/JS. No build step. The served site is the `public/` folder.

## Architecture

```
public/  ──git push──►  GitHub repo  ──┬──► Cloudflare Pages  (PRIMARY,  *.pages.dev)
                                       └──► GitHub Pages       (backup,   *.github.io/<repo>)
```

Both redeploy automatically on every push to `main`. Cloudflare Pages is the
URL you share; GitHub Pages is a zero-config fallback via the Actions workflow
in `.github/workflows/deploy-pages.yml`.

## Repo layout

| Path                  | Purpose                                                        |
|-----------------------|----------------------------------------------------------------|
| `public/`             | The entire served site. Cloudflare output dir + Pages artifact.|
| `public/_headers`     | Cloudflare Pages security/cache headers (GitHub Pages ignores). |
| `public/_redirects`   | Cloudflare Pages redirect rules (GitHub Pages ignores).         |
| `public/404.html`     | Custom 404 — honored by both platforms.                         |
| `public/.nojekyll`    | Stops GitHub Pages running Jekyll on `_`-prefixed files.         |
| `.github/workflows/`  | GitHub Pages backup deploy.                                     |

**Use relative asset paths** (`assets/x.css`, not `/assets/x.css`). Cloudflare
serves at a domain root, but GitHub Pages serves under `/<repo>/`. Relative
paths work in both; root-absolute paths break on GitHub Pages.

## Local preview

```sh
python3 -m http.server -d public 8000   # → http://localhost:8000
```

## One-time setup

### 1. Create the GitHub repo & push

```sh
git init -b main
git add .
git commit -m "Initial static site scaffold"
gh repo create hsyvy --public --source=. --push
```

### 2. Enable GitHub Pages (backup) — one-time

The workflow has `actions/configure-pages` with `enablement: true`, but the
repo's Actions `GITHUB_TOKEN` is usually **not** allowed to create a Pages
site (`Resource not accessible by integration`). So enable it once with an
owner-scoped token — either:

```sh
gh api -X POST repos/<owner>/hsyvy/pages -f build_type=workflow
```

or the dashboard: **Settings → Pages → Build and deployment → Source:
GitHub Actions**. After that the workflow publishes `public/` to
`https://<user>.github.io/hsyvy/` on every push. (Already done for this repo.)

### 3. Cloudflare Pages (primary) — one-time

`.github/workflows/deploy-cloudflare-pages.yml` runs `wrangler pages deploy`
on every push, which **deterministically** creates/updates the `hsyvy` Pages
project → `https://hsyvy.pages.dev` (no account-subdomain segment). The
dashboard "Connect to Git" flow is avoided because it now provisions a
*Worker* (`*.workers.dev`) instead of Pages.

It needs two repo secrets:

1. **API token** — Cloudflare dash → My Profile → API Tokens → Create Token →
   template *"Cloudflare Workers"* (or custom: Account · Cloudflare Pages ·
   Edit). Copy it.
2. **Account ID** — Cloudflare dash → Workers & Pages → right sidebar.
3. Add both:

   ```sh
   gh secret set CLOUDFLARE_API_TOKEN
   gh secret set CLOUDFLARE_ACCOUNT_ID
   ```
