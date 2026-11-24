# Atlassian MCP usage

- The Atlassian site is `one-atlas-mesa.atlassian.net`, cloudId `9c9e7af6-d0bb-4118-863c-21a32a2dfdaf`.
- Always pass `cloudId` on every Atlassian MCP call (`search`, `searchJiraIssuesUsingJql`, `getJiraIssue`, `searchConfluence`, `executeRead`, etc.). It is required, and omitting it causes "Input validation error: cloudId: Required".
- Do not call `getAccessibleAtlassianResources` first when the cloudId above will do; use it only if the site is different or the ID fails.
- `search` requires `cloudId` and `query`. `limit` must be between 10 and 30.
- `getConfluenceContent` takes `content_id` or `content_url` (not `pageId`). Use `detail: "full"` and `content_format: "markdown"` to read a page body.
- Jira web links (remote links) come from `executeRead` with `name: "listJiraIssueRemoteIssueLinks"`; issue-to-issue links are in `fields.issuelinks` from `getJiraIssue`.
- Project keys: ABHELP (help portal requests), ABIDEA (ideas), ABDAO (delivery work).

# Jira workflow skill (jira-cli.mjs / twg) and the sandbox

- `jira-cli.mjs` and `twg` commands (from the `jira:jira-workflows` skill and the other `twg-*` skills) need to reach `one-atlas-mesa.atlassian.net`, which the Bash sandbox blocks ("deny network-outbound one-atlas-mesa.atlassian.net:443").
- Run these commands with `dangerouslyDisableSandbox: true` from the first attempt. Do not run them in the sandbox first and wait for the violation.
- This applies only to `jira-cli.mjs` and `twg` commands that talk to Atlassian. Run every other Bash command in the sandbox as usual.

# Creating Confluence pages

- The Alderbank Confluence space is `ABDAO` ("Alderbank Digital Account Opening"), spaceId `2752516`. Its homepage id is `2752793`.
- `createConfluenceContent` does NOT take `spaceId`, `contentFormat` or a string `body`. Using them causes "Input validation error: parent: Required, contentType: Required, body: Expected object, received string". Use this shape:
  - `cloudId`: the cloudId above (top-level).
  - `parent`: an object, e.g. `{"spaceId": "2752516"}` for the space root, or `{"spaceId": "2752516", "parentContentId": "<page id>"}` to nest under a page.
  - `contentType`: `"page"` (classic page; matches existing pages in the space) or `"live_doc"`.
  - `title`: string, 1-255 characters.
  - `body`: an object, `{"format": "markdown", "value": "<content>"}`. Use `"html"` instead for panels, layouts or other rich elements.
- Example: `createConfluenceContent({ cloudId, parent: {"spaceId": "2752516"}, contentType: "page", title: "...", body: {"format": "markdown", "value": "..."} })`.
- If the tool schema is not loaded, call `ToolSearch` with `select:mcp__9c4c2359-55a7-4cb7-9d85-66e99475f541__createConfluenceContent` first.
- `updateConfluenceContent` and other write tools may also take object parameters. If a call fails validation, read the schema in the error message and correct the call rather than retrying with the same arguments.

# Publishing specs to Confluence

- Whenever you finish drafting a new spec (or similar document), offer to publish it to Confluence before ending your turn. Present it as an explicit option, e.g. via `AskUserQuestion` with "Publish to Confluence" and "Not now". Do not publish without the user's yes.
- Default space: Alderbank Digital Account Opening (`ABDAO`, spaceId `2752516`), unless the user names a different space. If they do, look it up with `executeRead` `listConfluenceSpaces` rather than guessing.
- Publish using the `createConfluenceContent` shape in "Creating Confluence pages" above. Keep "Spec (draft)" in the title and a "Status: Draft" line until the user says otherwise.
- After publishing, give the page link and say whether the rendering was checked.
