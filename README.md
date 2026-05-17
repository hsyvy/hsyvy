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

### 2. GitHub Pages (backup) — automatic

No action needed. The workflow uses `actions/configure-pages` with
`enablement: true`, so the first run enables Pages itself and publishes
`public/` to `https://<user>.github.io/hsyvy/`.

### 3. Connect Cloudflare Pages (primary)

Cloudflare dashboard → **Workers & Pages → Create → Pages → Connect to Git** →
pick the `hsyvy` repo, then:

| Setting                 | Value     |
|-------------------------|-----------|
| Framework preset        | `None`    |
| Build command           | *(empty)* |
| Build output directory  | `public`  |
| Production branch        | `main`    |

Save & Deploy → live at `https://hsyvy.pages.dev` (auto-deploys on push).

## Adding a custom domain later

DNS-only change, no code change (paths are already relative):

1. Cloudflare Pages project → **Custom domains → Set up a domain** → enter it.
2. If the domain is on Cloudflare DNS, the `CNAME` is added automatically;
   otherwise add `CNAME <name> hsyvy.pages.dev`.
3. Leave GitHub Pages on its `*.github.io` URL as the backup (a custom domain
   can only point at one host at a time — Cloudflare wins as primary).
