# Domain and GitHub Pages setup

Point `stableark.org` at this repository via GitHub Pages.

## 1. Push these site files

Ensure `index.html` and `CNAME` are on `main` in [stableark/stableark](https://github.com/stableark/stableark).

## 2. Enable Pages on GitHub

1. Open https://github.com/stableark/stableark/settings/pages
2. Under **Build and deployment** → **Source**, choose **Deploy from a branch**
3. Branch: `main` / folder: `/ (root)` → **Save**
4. Under **Custom domain**, enter `stableark.org` → **Save**
5. Wait for DNS check. After it is green, enable **Enforce HTTPS**

## 3. Set repository homepage

1. Open https://github.com/stableark/stableark
2. Click the gear next to **About**
3. Website: `https://stableark.org` → Save

## 4. Namecheap DNS (Advanced DNS)

Remove any existing URL Redirect / Parking / conflicting `@` or `www` records for this domain.

Add:

| Type | Host | Value | TTL |
| --- | --- | --- | --- |
| A Record | `@` | `185.199.108.153` | Automatic |
| A Record | `@` | `185.199.109.153` | Automatic |
| A Record | `@` | `185.199.110.153` | Automatic |
| A Record | `@` | `185.199.111.153` | Automatic |
| CNAME Record | `www` | `stableark.github.io.` | Automatic |

Optional IPv6 (AAAA) for `@`:

- `2606:50c0:8000::153`
- `2606:50c0:8001::153`
- `2606:50c0:8002::153`
- `2606:50c0:8003::153`

Do **not** use Namecheap “URL Redirect” at the same time as these A records.

## 5. Brand assets (site + GitHub)

Files live in [`assets/`](assets/). Site favicon / OG tags are already wired in `index.html`.

| Asset | Purpose |
| --- | --- |
| `mark.png` | Favicon + header mark |
| `og.png` | Website Open Graph / Twitter cards |
| `logo.png` | README / wordmark |
| `avatar.png` | GitHub org avatar |
| `github-social.jpg` | Repo **Social preview** (under 1 MB; GitHub’s limit) |

### Already done via API / push

- Pages deployed from `main` with `CNAME` → `stableark.org`
- Repo **Website** homepage set to `https://stableark.org`
- Repo description set

### Manual (GitHub has no public API for these)

1. **Org avatar:** [Organization profile settings](https://github.com/organizations/stableark/settings/profile) → upload [`assets/avatar.png`](assets/avatar.png)
2. **Repo social preview:** [Repository settings](https://github.com/stableark/stableark/settings) → **Social preview** → Edit → upload [`assets/github-social.jpg`](assets/github-social.jpg)

After upload, repo links on Slack/X/etc. should unfurl the ark banner; the org should show the cream-on-green ship avatar.

## 6. Verify

After DNS propagates (often minutes, sometimes up to 24–48h):

```bash
dig stableark.org +short
# expect the four 185.199.* addresses

curl -I https://stableark.org
curl -I https://stableark.org/assets/og.png
curl -I https://stableark.org/assets/mark.png
```

Official reference: [Managing a custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).

