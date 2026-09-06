# gaelvaldivieso.com

Personal portfolio / CV. One self-contained static page — no build step, no framework.

## Structure

| File | What it is |
|---|---|
| `index.html` | The whole site (HTML + inline CSS + inline JS). Edit this. |
| `favicon.svg`, `favicon-32.png`, `favicon.ico`, `apple-touch-icon.png` | Tab / home-screen icons |
| `og.png` | 1200×630 social share card (LinkedIn / Slack / WhatsApp previews) |
| `site.webmanifest`, `robots.txt`, `sitemap.xml` | Standard site metadata |
| `vercel.json` | Clean URLs + security headers |

## Make a change

```bash
# edit index.html, then:
git add -A
git commit -m "describe the change"
git push
```

Vercel auto-deploys `main` in ~30 s. Live at https://gaelvaldivieso.com

## Preview locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Regenerate the social image

Edit the source SVG, then rasterise with macOS `sips`:

```bash
sips -s format png og.svg --out og.png          # source SVG kept in git history / scratch
sips -s format png -Z 32 favicon.svg --out favicon-32.png
```

## Deploy setup (for reference)

- Host: **Vercel** (project imported from this GitHub repo, framework preset "Other", no build command).
- Registrar + DNS: **Cloudflare** (`gaelvaldivieso.com`).
- DNS records: `A @ → <Vercel IP>` and `CNAME www → cname.vercel-dns.com`, both **DNS-only** (grey cloud).
- SSL: automatic (Let's Encrypt, managed by Vercel).
