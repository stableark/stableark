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

## 5. Verify

After DNS propagates (often minutes, sometimes up to 24–48h):

```bash
dig stableark.org +short
# expect the four 185.199.* addresses

curl -I https://stableark.org
```

Official reference: [Managing a custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).
