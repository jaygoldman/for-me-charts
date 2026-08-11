# For / Not For Me

Make your own **life-preference charts** — the ones that show how much some thing is *for you* across your lifetime. Give it a title and an emoji, drag the line into shape, and download a PNG.

Inspired by [Ty from the Internet — *"Not Everything Is For You"*](https://tyfromtheinternet.com/not-everything-is-for-you/). All concept credit to Ty; this is just a free tool for making your own.

**Live:** https://formecharts.jaygoldman.com

## What it does

- **Draw by dragging.** The line starts flat at 50%. Drag the handle at each grid line up or down and the curve redraws in real time.
- **Sharp or smooth.** The line is a smooth hand-drawn curve by default. **Click a handle** to toggle it into a sharp corner for abrupt changes.
- **Make it yours.** Title, emoji, max age, axis labels, detail (number of handles), and colors (background, grid, line, text).
- **Presets** to start from (caffeine, naps, roller coasters, mosh pits, vegetables).
- **Export** as PNG (2×), crisp SVG, or copy the image straight to your clipboard.
- **Share** a link — the entire chart is encoded in the URL, so anyone who opens it sees your exact chart. Autosaves to your browser so a refresh never loses work.

Everything runs in the browser. Nothing is uploaded; there is no backend.

## Run locally

It's a single self-contained file — just open it:

```bash
open index.html
```

(Or serve the folder with anything, e.g. `python3 -m http.server`, if your browser restricts `file://` clipboard APIs.)

## Project layout

| File | Purpose |
| --- | --- |
| `index.html` | The entire app — inline CSS + JS, no dependencies, no build step. |
| `favicon.svg` | Tab icon. |
| `wrangler.toml` | Cloudflare Pages project config for `wrangler pages deploy`. |
| `.github/workflows/deploy.yml` | Auto-deploy to Cloudflare Pages on merge to `main`. |

## Deploy to Cloudflare Pages (zero cost)

There are two ways to deploy. **Pick one** — don't run both, or they'll fight over deployments.

### Option A — Connect the Git repo (simplest, recommended)

1. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**.
2. Choose the `jaygoldman/for-me-charts` repo.
3. Build settings:
   - **Framework preset:** `None`
   - **Build command:** *(leave blank)*
   - **Build output directory:** `/`
4. **Save and Deploy.**

Cloudflare now rebuilds on every push to `main` and creates a **preview URL for every pull request** automatically — no secrets, no Actions. If you use this option, you can delete `.github/workflows/deploy.yml`.

### Option B — Deploy via GitHub Actions (CI-controlled)

The included workflow (`.github/workflows/deploy.yml`) deploys with Wrangler on merge to `main`. One-time setup:

1. Create the Pages project once:
   ```bash
   npx wrangler pages project create for-me-charts --production-branch main
   ```
2. Add two **GitHub repo secrets** (Settings → Secrets and variables → Actions):
   - `CLOUDFLARE_API_TOKEN` — a token with the **Cloudflare Pages: Edit** permission ([create one](https://dash.cloudflare.com/profile/api-tokens)).
   - `CLOUDFLARE_ACCOUNT_ID` — your account ID (right sidebar of any Cloudflare dashboard page).

Merges to `main` now deploy to production; PRs get a preview deployment.

### Custom domain — `formecharts.jaygoldman.com`

Since `jaygoldman.com` is on Cloudflare, this is a few clicks:

1. Open the **for-me-charts** Pages project → **Custom domains** → **Set up a custom domain**.
2. Enter `formecharts.jaygoldman.com` → **Continue** → **Activate domain**.
3. Cloudflare automatically adds the `CNAME` record (`formecharts` → the project's `*.pages.dev` hostname) in the `jaygoldman.com` zone and provisions the SSL certificate. Live within a minute or two.

If the domain's DNS is **not** on Cloudflare, add a `CNAME` record for `formecharts` pointing at `for-me-charts.pages.dev` at your DNS provider, then complete the domain verification in the Pages dashboard.

## License

[MIT](LICENSE). Concept credit to [Ty from the Internet](https://tyfromtheinternet.com/not-everything-is-for-you/).
