---
name: deed-page
description: Publish a generated HTML file or static folder to a permanent public deed.page URL with one curl request, and update or delete it with the returned owner token. Use when the user wants a shareable link for a report, dashboard, deck, prototype, or docs folder and no existing deployment target applies.
---

# deed.page Publish

Turn a static artifact the user asked for into a live `https://<slug>.deed.page/` link. deed.page needs no account or API key; the first publish returns an owner token that is the only way to update or delete the site. Adapted from the MIT-licensed [f-o-x11/deed-page-skill](https://github.com/f-o-x11/deed-page-skill); the full API is at https://deed.page/docs.

## Delegation

Run in the main conversation by default. Delegation can increase usage: obtain explicit approval for the proposed agent count and scope before using subagents. Reuse that approval within its bounds; ask again before expanding the approved count or scope.

## Decide whether it fits

- Use it when the user wants a link to something static the agent produced and has not named another host. If the project already deploys somewhere (Vercel, Netlify, GitHub Pages, a VPS), follow that operating contract instead.
- Free sites are public. Do not publish secrets, credentials, customer data, or anything the user has not agreed to make public.
- deed.page serves static files only: no server-side code, databases, or host-side builds. Build locally first and publish the output folder.

## Publish

Choose a slug of lowercase letters, digits, and hyphens (1-63 characters) with a random suffix, then upload. A single file is stored as `index.html`; a folder is sent as a tarball whose root contains `index.html`.

```bash
SLUG="report-$(head -c4 /dev/urandom | od -An -tx1 | tr -d ' \n')"
curl -sS -T report.html "https://deed.page/v1/sites?slug=$SLUG" -H "X-Client: <agent name>" ${DEEDPAGE_TOKEN:+-H "Authorization: Bearer $DEEDPAGE_TOKEN"}

tar -C ./dist -czf /tmp/site.tar.gz .
curl -sS -T /tmp/site.tar.gz "https://deed.page/v1/sites?slug=$SLUG" -H "X-Client: <agent name>" ${DEEDPAGE_TOKEN:+-H "Authorization: Bearer $DEEDPAGE_TOKEN"}
```

The JSON response carries `url` and, on a first publish without a token, `token`. Keep the token in `DEEDPAGE_TOKEN` for the session, tell the user to store it safely, and never write it into a page, commit, or log. It cannot be recovered unless the owner attached an optional recovery email.

## Update, delete, recover from errors

- Update by rerunning the same publish with `DEEDPAGE_TOKEN` set; each publish becomes a new version at the same URL.
- Delete only when the user asks: `curl -sS -X DELETE "https://deed.page/v1/sites/$SLUG" -H "Authorization: Bearer $DEEDPAGE_TOKEN"`.
- `409 slug_taken`: use the returned `suggestion` or a new suffix. `400 slug_invalid`: fix the slug (it is never rewritten). `429`: wait `retry_after` seconds. `422 content_rejected`: remove the flagged file and do not retry.
- Free plan limits: 25 sites, 25 MB per site, 4 MB per single request. Larger uploads use the manifest flow in the docs.

## Done when

The returned `url` answers with HTTP 200 and shows the expected content, the user has the link, and the user knows where the owner token is and that it is needed for future updates.
