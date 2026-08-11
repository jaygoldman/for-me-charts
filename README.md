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
| `wrangler.toml` | Cloudflare Pages project config (used if you deploy via Wrangler/CI). |

## Deploy to Cloudflare Pages (zero cost)

This project deploys as a **Cloudflare Worker using Static Assets** — there is no Worker script; `wrangler deploy` uploads the repo directory (`[assets]` in `wrangler.toml`) and Cloudflare serves the files directly. Static-asset requests are free.

Connect the repo once (Workers Builds):

1. Cloudflare dashboard → **Workers & Pages** → **Create** → **Import a repository** → choose `jaygoldman/for-me-charts`.
2. Leave the **build command** blank and the **deploy command** as **`npx wrangler deploy`** (the default). `wrangler.toml` supplies the project name and assets directory.
3. **Save and Deploy.** Every push to `main` redeploys; pull requests get preview URLs.

> **Deploy command must be `npx wrangler deploy`, not `npx wrangler pages deploy`.** This is a Worker (Static Assets) project, not a classic Pages project, so the Pages command would target a different, non-existent resource.

### Custom domain — `formecharts.jaygoldman.com`

Since `jaygoldman.com` is on Cloudflare, attach the domain to the Worker:

1. Open the **for-me-charts** Worker → **Settings → Domains & Routes** → **Add → Custom domain**.
2. Enter `formecharts.jaygoldman.com` → **Add domain**.
3. Cloudflare adds the DNS record in the `jaygoldman.com` zone and provisions the SSL certificate automatically. Live within a minute or two.

## License

[MIT](LICENSE). Concept credit to [Ty from the Internet](https://tyfromtheinternet.com/not-everything-is-for-you/).
