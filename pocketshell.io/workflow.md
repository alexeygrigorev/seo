# Reproduce the PocketShell SEO fix

## Process used on October 1, 2026

We followed the Ahrefs email into the affected-URL table, cloned the site,
and compared its sitemap URLs with the canonical tags produced by the
shared blog template. This established that the sitemap already used the
correct trailing-slash URLs; the template and internal links needed fixing.

| Step | Action and evidence |
| --- | --- |
| Diagnose | Capture the thirteen HTTP-200 sitemap URLs and their slashless HTTP-301 canonical targets. |
| Implement | Change shared URL generation and internal links; shorten eight descriptions and two search titles. |
| Validate locally | Build with the pinned rustkyll release; run the generated-site checker. It failed on the old output and passed on the new output. |
| Preview | Open the blog index and a post in Chrome; inspect DOM metadata, card links, and visible headings. |
| Review | Push `fix/ahrefs-canonical-urls` and open PR #3. Its Linux CI build passed. |
| Release | Merge the checked PR after the user requested it; update the local `main` checkout and wait for the Pages deployment. |
| Check production | Fetch the live sitemap and every listed page without following redirects, then run the same checker against the downloaded HTML. |
| Recrawl | Start a new Ahrefs crawl after deployment and inspect the completed comparison with the original crawl. |
| Record | Save before/after metadata, live HTTP results, resolved issue counts, and a screenshot; commit and push this SEO repo. |

The source fix was commit `c79f83ba104bd43a68011f2d624b0e2d121f7081`.
PR #3 merged as `bab70acf26790891b7bcd721e95ff76971130cc0`.
Deployment run `36919198209` succeeded. The completed recrawl
`01-10-2026T221021P0200` raised the health score from 73 to 100 and cleared
the targeted issues. Full evidence is in the
[dated audit record](audits/2026-10-01/README.md).

For the next alert, follow the sections below and use a new branch and
dated audit folder. Keep the October 1 evidence as a historical record.

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

The example branch above already exists from this fix. For future work,
start a new branch from an up-to-date `main` with a name matching the new
issue. Preserve any existing local changes before switching branches.

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

Linux amd64, with Node.js available:

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

The release commands used for this fix, from `~/git/pocketshell-site`, were:

```sh
gh pr checks 3 --repo PocketShell-io/pocketshell-site
gh pr view 3 --repo PocketShell-io/pocketshell-site --json state,mergeable,mergeStateStatus,headRefOid
gh pr merge 3 --repo PocketShell-io/pocketshell-site --merge --match-head-commit c79f83ba104bd43a68011f2d624b0e2d121f7081
git switch main
git pull --ff-only origin main
gh run list --repo PocketShell-io/pocketshell-site --branch main --limit 3 --json databaseId,status,conclusion,headSha,url
gh run view 36919198209 --repo PocketShell-io/pocketshell-site --json status,conclusion,jobs
```

These IDs document the completed release. For another PR, substitute its
number, reviewed head commit, and deployment run ID. Check that the
deployment's `headSha` matches the new merge commit; an older successful
deployment does not prove the new changes are live. `gh pr merge` updates
remote `main` and triggers deployment, so an additional source push after
merging is unnecessary. The separate SEO documentation commit still needs
its own push.

Then verify the live blog index and posts declare self-canonicals and that
those URLs return 200 without redirecting. Verify the live sitemap agrees.
From the SEO repository, the site-specific live check downloads a snapshot
and runs the checker from the PocketShell source checkout:

```sh
node pocketshell.io/scripts/verify-live.mjs ../pocketshell-site work/pocketshell-live pocketshell.io/audits/2026-10-01/live-verification.json
```

Choose a new dated report path for future audits. It follows no redirects
and preserves the downloaded HTML in the specified snapshot directory.
Start a fresh crawl at
[PocketShell Site Audit](https://app.ahrefs.com/site-audit/10427569/overview).
Wait for completion and inspect both canonical issue counts and the
description/title issues. Record the new crawl ID and remaining findings.
Do not treat a local build or an old crawl as live proof.

In Ahrefs, click **New crawl** on the PocketShell overview. The interface
switches to **Crawl log** and progresses through Crawling and Finalizing.
Wait until **Project history** shows Completed. This crawl took 2m 18s.
Then open **Overview** for the health score and **All issues → All tracked**
for explicit zero counts and removed-URL counts. The Actual filter hides
resolved issues, so it is insufficient for recording their before/after
counts. Compare with the original crawl, not another project's audit.

For this run, both canonical errors went from 13 to 0, long descriptions
from 8 to 0 across indexable/non-indexable groups, long titles from 2 to 0,
and pages linking to redirects from 14 to 0. Ahrefs also reported exactly
13 canonical changes, eight description changes, and two title changes.
Record remaining warnings and notices separately; health score 100 did
not mean every notice had disappeared.

## Update this SEO repository

Add a dated record below `pocketshell.io/audits/`. Include the baseline,
cause, changed files, before/after metadata, source commit and PR, exact
validation, deployment status, and recrawl outcome. Update the project
README status. Commit and push this SEO repository after reviewing the
files for credentials and private data. Keep each site's records in its
own folder.

For the October 1 audit we saved:

- `ahrefs-baseline.txt`: original thirteen-page issue table.
- `changes.json`: source commits and per-page before/after metadata.
- `blog-preview.png`: local visual check.
- `live-verification.json`: production HTTP statuses and checker result.
- `ahrefs-resolved-issues.json`: completed recrawl counts and remaining findings.
- `ahrefs-health-100.png`: final overview screenshot.

From `~/git/seo`, review and publish only the intended PocketShell records:

```sh
git diff --check
git diff -- pocketshell.io
git add pocketshell.io
git commit -m "Document PocketShell SEO maintenance process"
git push origin main
git status --short
git rev-parse HEAD
git ls-remote origin refs/heads/main
```

Verify that the local and remote commit hashes match. Include untracked
files in the review before staging, and keep downloaded HTML snapshots in
scratch storage rather than adding them to the public repository.
