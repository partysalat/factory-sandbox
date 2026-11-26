---
type: grilling
status: closed
blocked-by: [04-api-contract]
claimed-by: Ben Lenser
---
## Question

How is the SolidJS app structured?
Decide the routes (list, post, admin list, admin editor), the router, data fetching, and the admin editor behavior.
Decide where markdown renders (browser or Lambda) and how output is sanitized.

## Resolution

- Router: `@solidjs/router`. Routes: `/` (public list), `/posts/:slug` (public post), `/admin` (admin list, shows the login screen when no token is in `sessionStorage`), `/admin/new`, `/admin/edit/:slug`, `*` (not found).
- Data: one small typed `fetch` wrapper in `web/src/api.ts`. Components use `createResource`. No query library. The wrapper adds the Bearer header for admin calls. A `401` on any admin call clears the token and routes to the login screen. Loading and error states are plain inline text. An API `404` shows the not-found page.
- Markdown: render in the browser with `marked`, sanitize with `DOMPurify`. One `renderMarkdown()` serves the public post page and the admin preview. The API contract is unchanged. No server-side HTML, so crawlers and link previews see an empty shell. The page sets `document.title` per route.
- Tag policy: `DOMPurify` forbids `img`, `iframe`, `script`, because images are out of scope.
- CSP: `default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; connect-src 'self'; base-uri 'self'; frame-ancestors 'none'`. This amends the edge routing ticket: CDK defines one custom response headers policy with the same security headers as the managed one, plus this CSP.
- Admin editor: title field. Slug field on create only (derives from the title, editable, read-only on edit). Markdown `textarea` with a live preview pane. Buttons: Save, Publish or Unpublish, Delete (confirm dialog). No autosave. Warn on leaving with unsaved changes. The login screen validates the token with `GET /api/admin/posts`.
- Styling: plain CSS with Vite (global stylesheet plus CSS modules), no UI framework, system fonts, one responsive column, `prefers-color-scheme` dark mode. Dates use `Intl.DateTimeFormat` in the reader's locale.
