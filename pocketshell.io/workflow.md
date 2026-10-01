# Reproduce the PocketShell SEO fix

## Locate the alert

In the signed-in Gmail session, find the Ahrefs email named
`(Pocketshell) Non-canonical page in sitemap: 13 URLs`. Follow its crawl or
issue links. The baseline crawl is `01-10-2026T140657` in project `10427569`.
Read the affected-page and canonical columns before editing. Capture public
URLs and counts, without copying email account details or unsubscribe links.

The original report had a health score of 73, with 30 internal URLs analyzed.
Its main issue pairs were 13 non-canonical sitemap pages and 13 canonicals
pointing to redirects. It also reported 16 redirects, pages linking to
redirects, eight long descriptions, and two long titles. Some redirects are
expected (HTTP to HTTPS and old app bookmarks); they are not all defects.

## Clone and inspect the actual deployment

```sh
cd ~/git
git clone git@github.com:PocketShell-io/pocketshell-site.git
cd pocketshell-site
git switch -c fix/ahrefs-canonical-urls
```

If the checkout exists, inspect its status before pulling or switching
branches. Read any applicable `AGENTS.md`, then `_config.yml`, `Makefile`,
and `.github/workflows/deploy.yml`. This project has Jekyll-style sources,
but production uses **rustkyll**, not a Ruby Jekyll build.

Key files:

| File | Purpose |
| --- | --- |
| `_config.yml` | Public origin and `/blog/:title/` permalink |
| `blog.md` | Blog index permalink `/blog/` |
| `_includes/head.html` | Blog canonical, social metadata, JSON-LD |
| `_layouts/landing.html` | Homepage metadata and post cards |
| `_layouts/blog-index.html`, `_layouts/post.html` | Post card links |
| `_posts/*.md` | Descriptions, titles, and Markdown cross-links |
| `app.md`, `login.md` | Intentional noindex redirects to the app subdomain |
| `scripts/check-seo.mjs` | Generated-site checker added by PR #3 |

## Apply the URL convention

Use `site.url | append: page.url` for canonical URLs. Reuse that value in
`og:url` and JSON-LD. Use `p.url` in cards instead of constructing a URL
from `p.slug`. In Markdown, link directly to `/blog/<slug>/`.

The sitemap was already correct. Changing it to slashless URLs would make
it list redirects. Preserve `/blog/` and `/blog/:title/` as the canonical
routes. Old slashless bookmarks can continue to redirect.

Keep metadata descriptive and specific. For this audit, shorten the
homepage and seven post descriptions to at most 160 characters. Set
`seo_title` on the tmux and aplexer-crash posts to shorten their search
titles while retaining the visible heading. Escape title and description
values in HTML attributes. The 160/70 character limits are site editorial
checks, not promises about search results or rankings.

## Build and validate

Linux/macOS, with Node.js available:

```sh
make install
make check
```

The Makefile defaults to the Linux amd64 binary. On another platform,
select the matching rustkyll release asset. The Windows commands used here
(PowerShell, with GitHub CLI and Node.js installed) were:

```powershell
New-Item -ItemType Directory -Force .bin | Out-Null
gh release download v0.5.3 --repo alexeygrigorev/rustkyll `
  --pattern rustkyll-windows-amd64.exe --dir .bin
./.bin/rustkyll-windows-amd64.exe build
node scripts/check-seo.mjs
```

Reuse an existing matching binary rather than downloading over it. The
downloaded Windows binary's SHA256 was
`f7dfa13a485a774fef2db5904512f9e7fbeb6164c43e7ceadc2a9155b4482457`.

The checker reads `_site/sitemap.xml` and the generated HTML. It checks
self-canonicals, Open Graph URLs, JSON-LD URLs, metadata limits, matching
social descriptions, noindex exclusion, and internal blog links. The
October 1 fixed output passes for 14 sitemap pages, 13 blog pages, and
121 internal blog links. The original output fails these checks.

For a visual preview:

```powershell
./.bin/rustkyll-windows-amd64.exe serve --no-watch
```

Open `http://127.0.0.1:4000/blog/` and a post. Check the actual DOM metadata
and card links. The local preview should still declare production canonical
URLs; it should not declare localhost as canonical.

## Publish and confirm live resolution

Commit the source changes, push the branch, and open a PR. The `Check site`
workflow validates the production Linux builder. After merging, wait for
`Deploy site` to finish successfully. Deployment also runs `make check`.

Then verify the live blog index and posts declare self-canonicals and that
those URLs return 200 without redirecting. Verify the live sitemap agrees.
Start a fresh crawl at
[PocketShell Site Audit](https://app.ahrefs.com/site-audit/10427569/overview).
Wait for completion and inspect both canonical issue counts and the
description/title issues. Record the new crawl ID and remaining findings.
Do not treat a local build or an old crawl as live proof.

## Update this SEO repository

Add a dated record below `pocketshell.io/audits/`. Include the baseline,
cause, changed files, before/after metadata, source commit and PR, exact
validation, deployment status, and recrawl outcome. Update the project
README status. Commit and push this SEO repository after reviewing the
files for credentials and private data. Keep each site's records in its
own folder.
