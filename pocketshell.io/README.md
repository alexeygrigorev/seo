# PocketShell SEO maintenance

- Site: https://pocketshell.io
- Source: [PocketShell-io/pocketshell-site](https://github.com/PocketShell-io/pocketshell-site)
- Local checkout: `~/git/pocketshell-site`
- Build: rustkyll v0.5.3, using Jekyll-compatible Liquid templates and Markdown.
- Deployment: GitHub Pages, triggered by pushes to `main`.
- Ahrefs project: `10427569`, named Pocketshell.

The October 1, 2026 alert reported **13 non-canonical pages in the sitemap**
and **13 canonicals pointing to redirects**. The blog index and twelve posts
were served at trailing-slash URLs but declared slashless canonical URLs.
GitHub Pages redirects the slashless variants to the trailing-slash pages.

[PR #3](https://github.com/PocketShell-io/pocketshell-site/pull/3) fixes the
shared canonical, Open Graph, and structured-data URLs, internal blog links,
eight long descriptions, and two long search titles. It adds a generated-site
checker to pull-request CI and the deployment build.

PR #3 is merged at `bab70acf26790891b7bcd721e95ff76971130cc0`. The
[Pages deployment](https://github.com/PocketShell-io/pocketshell-site/actions/runs/36919198209)
succeeded. All 14 live sitemap pages return HTTP 200 without redirects and
pass the generated-site SEO checker. The completed post-deployment Ahrefs
crawl reports a **health score of 100 (up from 73)** and zero errors. The
original canonical, description-length, title-length, and redirect-link
issues now have zero affected URLs. Three redirect warnings and other
notices remain; see the audit record.

- [Repeatable workflow](workflow.md)
- [October 1 audit and validation](audits/2026-10-01/README.md)
- [Structured before/after changes](audits/2026-10-01/changes.json)

This folder documents this site's SEO work. The implementation and automated
checker live in the PocketShell source repository.
