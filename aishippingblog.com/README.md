# aishippingblog.com — Alexey On Data

SEO maintenance for [aishippingblog.com](https://aishippingblog.com/), hosted on Substack.

## Where to start next time

Read [the Substack workflow](workflow.md) before editing. It records the exact controls used, the separate SEO Save button, comment-page inheritance, and live verification.

Use the newest audit as a historical record; inspect the current page and current Ahrefs results before changing anything. Do not overwrite a description that has since been edited just because an older audit contains a replacement.

## October 1, 2026 result

The email **“(Aishippingblog) Meta description too short: 40 URLs”** led to 33 post SEO-description updates. Three comment pages inherited their parent posts' new descriptions, fixing 36 URLs in total.

The new descriptions are 134–157 characters. Visible post subtitles and article bodies were preserved. Every changed URL was checked against its live HTML meta description.

A fresh Ahrefs crawl completed and confirmed:

| Measure | Before | After |
| --- | ---: | ---: |
| Meta description too short | 40 | 4 |
| URLs with changed descriptions | — | 36 |

The remaining four URLs are generated Substack pages with no SEO editing controls found in the page UI, publication settings, or website editor:

- <https://aishippingblog.com/archive>
- <https://aishippingblog.com/sitemap>
- <https://aishippingblog.com/sitemap/2025>
- <https://aishippingblog.com/sitemap/2026>

These warnings remain open. No Ahrefs patches were published and no warnings were ignored.

## Files

- [Workflow](workflow.md)
- [Audit notes](audits/2026-10-01/README.md)
- [Exact before/after descriptions](audits/2026-10-01/meta-descriptions.json)
- [Human-readable change record](audits/2026-10-01/meta-description-changes.txt)

The records concern the short-description issue only. The original email also reported long descriptions, long titles, missing alt text, and other issues; those were outside this work's scope.
