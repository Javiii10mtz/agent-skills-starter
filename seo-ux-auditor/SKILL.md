---
name: seo-ux-auditor
description: Audits a web page in the browser or via HTTP to verify basic SEO metadata (title, meta description, canonical, single H1, Open Graph) and detect broken links or failing assets (4xx/5xx), returning a concise checklist report with fixes.
---

# SEO & Link Auditor (Starter Edition)

A lightweight, production-ready Agent Skill for **Antigravity**, **Claude Code**, and **Cursor** that checks the two highest-impact technical SEO fundamentals on any live URL or `localhost` preview:
1. **Basic SEO metadata** (`<title>`, `<meta name="description">`, `<link rel="canonical">`, `<h1>` count, and Open Graph basics).
2. **Broken links & failing assets** (HTTP `4xx` / `5xx` responses on internal links and images).

> Looking for full desktop + mobile visual UX audits, WCAG contrast checks, responsive overflow detection, and Tailwind CSS fix snippets? Check out the full **AI Agent Power Pack v1** (includes `seo-ux-auditor` Pro, `structured-web-extractor`, and `landing-page-shipper`).

---

## When to Use

Activate this skill when the user asks to:
- Run a quick pre-launch metadata check on a website or landing page.
- Find broken internal links (`404`, `500`) or missing image `alt` attributes.
- Validate that a page has a clean `<title>`, meta description, and canonical URL.

---

## Workflow

### 1. Fetch & Inspect the Page
1. Confirm the target URL (e.g., `https://example.com` or `http://localhost:3000`).
2. Open the page in the browser (or fetch rendered HTML if it is a static/SSR site).
3. Extract the `<head>` metadata and all `<a>` / `<img>` elements from the DOM.

### 2. Run the Starter Audit Checklist

#### A. Basic Metadata
- `<title>`: Present, non-empty, and ideally between **50–60 characters**.
- `<meta name="description">`: Present, non-empty, and ideally between **140–160 characters**.
- `<link rel="canonical">`: Present and points to the expected canonical URL.
- `<h1>`: Exactly **one** visible `<h1>` on the page.
- **Open Graph**: Check for `og:title`, `og:description`, and `og:image`.

#### B. Broken Links & Image Alt Check
- Collect unique `href` values from `<a href="...">` (skip `mailto:`, `tel:`, and `#` anchors).
- Verify internal links (and key external links, capped at 30 requests per run to avoid rate limits) return HTTP `2xx` or `3xx`. Flag any `4xx`/`5xx` or connection errors.
- Check `<img>` tags for missing `alt` attributes (informative images should have descriptive `alt` text; decorative images should use `alt=""`).

### 3. Output the Report
Return a concise Markdown report with:
1. **Metadata Summary** (Pass / Warn / Fail table with character counts).
2. **Broken Links & Missing Alt List** (URL, HTTP status, and element location).
3. **Copy-Paste Fixes** for any missing `<head>` tags.

---

## Example Input & Output

### Input
> "Use `seo-ux-auditor` to check metadata and broken links on `https://example-saas.dev`."

### Output
```markdown
# Starter SEO & Link Audit — https://example-saas.dev

## 1. Metadata Check
| Tag | Status | Value / Notes |
| :--- | :--- | :--- |
| `<title>` | PASS (54 chars) | `ExampleSaaS — Automated Weekly Reports for Dev Teams` |
| `<meta name="description">` | FAIL (Missing) | No meta description found. |
| `<link rel="canonical">` | PASS | `https://example-saas.dev/` |
| `<h1>` count | WARN (2 found) | Merge or demote the second `<h1>` ("Pricing") to `<h2>`. |
| Open Graph | WARN | Missing `og:image`. |

## 2. Broken Links & Assets
- `https://example-saas.dev/docs/v1-quickstart` → **HTTP 404** (linked in footer)
- `<img src="/hero.png">` → Missing `alt` attribute

## 3. Recommended Fixes
```html
<meta name="description" content="Automate weekly engineering reports in 5 minutes with zero manual copy-pasting." />
<meta property="og:image" content="https://example-saas.dev/og-cover.png" />
```
```

---

## Error Handling

- **403 / CAPTCHA / Bot Protection**: Stop immediately and inform the user. Never attempt to bypass anti-bot gates.
- **Empty body on `curl` (Client-Side Rendered SPA)**: Switch to a real browser inspection tool and wait for the page to finish rendering before reading `<head>` and `<a>` tags.
- **Timeout on external links**: Mark as `TIMEOUT` rather than `404` so the user knows it may be a transient network issue.
