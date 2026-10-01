# Repeat the meta-description repair on aishippingblog.com

## Site and tools

- Publication: **Alexey On Data**
- Website: <https://aishippingblog.com/>
- Platform: Substack
- Ahrefs project: **Aishippingblog**, project ID **10427571**
- Current issues: <https://app.ahrefs.com/site-audit/10427571/issues>
- Publication settings: <https://aishippingblog.com/publish/settings>
- Website editor: <https://aishippingblog.com/publish/website-editor/home>

In this run, the in-app browser was signed out of Gmail. The user's Chrome browser already had the alert open and was signed in to Gmail, Ahrefs, and Substack. Using that existing session avoided login work. Browser session/tab IDs are temporary; discover fresh tabs next time.

For Codex browser work, read the installed browser skill and the selected browser's complete documentation first. Prefer a suitable connector if one is available for the operation; none was available for the email, Ahrefs, or Substack editing in this run. Use the supported browser runtime rather than reading cookies or making authenticated background requests.

## 1. Capture the current affected pages

1. Open the Ahrefs email and follow the **View** link in the **Meta description too short** row, or open the same issue from the current project audit.
2. Record the crawl date, URL, page title, current description, and character count for every result.
3. Separate post URLs (`/p/...`), comment URLs (`/p/.../comments`), and generated archive/sitemap URLs.
4. Deduplicate parent posts. Updating a parent post can repair its comment page too; verify that rather than editing the parent twice.

The original crawl analyzed 96 internal URLs. All 40 short-description results were present in one table, so pagination was unnecessary. Use the current result count next time rather than assuming it is still 40.

## 2. Draft from the actual article

Read each post before drafting its description. Use its subject and concrete content; avoid padding a short subtitle with generic claims.

For this issue, descriptions of roughly 130–160 characters worked well: every replacement ended up at 134–157. Substack's editor recommended 50–160, but Ahrefs flagged several descriptions near 99 characters as too short. Check the current Ahrefs issue definition rather than assuming Substack's minimum will clear the warning.

- Preserve the visible subtitle by editing the dedicated **SEO description** field.
- Keep each description accurate and specific to its article.
- For expired enrollment announcements, describe the original dated announcement instead of implying registration is still open.
- Keep the SEO title, post URL, publication date, audience, comments, tags, social preview, and article body unchanged.

## 3. Edit and save the SEO field

1. Open the public post on aishippingblog.com.
2. Open the post's three-dot menu beside **Share** and choose **Edit**.
3. Wait for the editor to load, then open **Settings**.
4. Expand **SEO Options**.
5. Replace **SEO description**. Its placeholder was `Enter a custom description...`.
6. Read the displayed character count and inspect the new text.
7. Click the **Save** button associated with the SEO description.
8. Wait for that save to finish before leaving the editor.

**Done alone does not save a pending SEO-field edit.** Early in this run, filling the description and clicking Done left the public meta description unchanged. The separate SEO Save button is essential. The toolbar's Saved indicator also does not prove a pending SEO setting has reached the public page.

There can be more than one Save button in Post settings, including a comments-settings Save. Inspect the current DOM and target the SEO form's Save. In the first-post editor, the SEO Save was the `button[type="submit"]`, while a second Save was a `button[type="button"]`. Do not apply that selector blindly to other pages.

After filling, allow the UI to render its new count and Save controls. In this run, separate fill/inspect and Save actions were reliable. If a click appears to do nothing, inspect the page state before trying again.

## 4. Verify the public HTML

Open the public URL in a separate tab after saving. The success criterion is the HTML `<meta name="description">` content matching the exact intended text, not just the settings textbox showing the replacement.

With an initialized Codex browser tab, this read-only DOM check was used:

```js
const description = await verificationTab.playwright.evaluate(
  () => document.querySelector('meta[name="description"]')?.content
);
```

If the old description appears, first allow the save to complete, then reload the public verification tab. Inspect whether the correct SEO Save actually ran. Do not count the URL as fixed until the public HTML matches.

For each changed post, record:

- URL and old description
- Replacement and character count
- Live verification result
- Verification date

Also open each affected `/comments` URL. All three inherited their parent's custom SEO description in this run. Record the comment URL as independently verified rather than merely assuming inheritance.

If Exit shows **You have unpublished changes**, inspect the pending state. Do not use Update or Discard just to dismiss it; these can affect changes outside the SEO field. The SEO-field Save in this workflow updated live metadata directly, without requiring a post-body Update or an email send.

## 5. Handle generated pages honestly

The archive and HTML sitemap pages did not expose per-page SEO-description controls. A search for SEO in publication settings returned no results; the website editor offered layout, colors, typography, and images, with no archive/sitemap SEO overrides.

Leave these as unresolved platform limitations unless a newly available control can change the live HTML. Recheck the controls next time, since the platform may change. Do not rename the publication, change its global description, hide pages, alter domains, deploy Ahrefs patches, or ignore warnings solely to reduce this issue count.

## 6. Re-crawl and record the outcome

1. After live verification, click **New crawl** in the Ahrefs project.
2. Wait for completion and open the current **All issues** report.
3. Check both **Meta description too short** and **Meta description changed**.
4. Open the remaining short-description results and record their exact URLs.
5. Save an audit note, before/after data, and screenshots in a dated folder under `audits/`.

For the October 1 run, the fresh crawl confirmed **4 short descriptions remaining** and **36 descriptions changed**. The remaining result table contained exactly the four generated pages listed in this project's README.

## Browser automation notes from this run

These locators describe the UI observed during this run. Inspect fresh page state before reusing them:

| Step | Observed control |
| --- | --- |
| Post menu | Last button in region `Post UFI`; menu included `Edit` |
| Open editor | Menu item `Edit` |
| Post settings | Button `Settings` |
| Expand SEO | Text `SEO Options` |
| Description field | Placeholder `Enter a custom description...` |
| Apply metadata | SEO field's `Save` button |
| Close settings | Button `Done` |

Use one reusable post-edit tab and a separate verification tab. Process articles serially so the current article, replacement, and verification stay aligned. Pass the pending URL/index and replacement explicitly into helper functions; avoid a helper capturing an earlier version of a mutable variable.

The final Ahrefs issue row changed after the comparison data loaded: it included the remaining count and change counts. Exact matching of the whole row text then failed. Match the row by its issue label, inspect it, and target its current count link rather than assuming its complete accessible name is only `Meta description too short 4`.

## What belongs in this public folder

Keep public URLs, public descriptions, workflow instructions, and audit evidence. Exclude cookies, credentials, tokens, private draft links, email contents, subscriber data, account exports, and machine-specific session IDs.
