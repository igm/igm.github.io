# igm.github.io

Source for [blog.igormihalik.com](https://blog.igormihalik.com) — plain HTML, no static site generator. What's in `master` is exactly what gets served.

## Adding a post

There is no generator, so a new post is a copy plus a handful of deliberate edits.
Steps 1–3 create the post; steps 4–6 are what makes it *appear* on the site.

**The shell is identical on every page and never changes when you publish.** It
carries no counts, no topic list, and no "currently" note — nothing that a new
post would invalidate. Adding a post touches only the files listed below, never
all of them.

### 1. Copy the template

```sh
cp -r post/_template "post/my-post-slug"
```

The directory name **is** the URL: `/post/my-post-slug/`.

### 2. Fill in the placeholders

All of them appear in the copy; search for them:

| Placeholder | Becomes | Appears |
|---|---|---|
| `POST TITLE` | the title | `<title>`, `og:title`, `<h1>` |
| `YOUR-SLUG` | the directory name | canonical URL, `og:url`, breadcrumb |
| `ONE-SENTENCE DESCRIPTION.` | meta description | `<meta name="description">` |
| `TOPIC` | a topic tag | the `.tag-cloud` in the header |
| `YYYY-MM-DD` | machine date | `<time datetime>`, `article:published_time` |
| `MONTH D, YYYY` | display date | the visible `<time>` |
| `N min read` | reading time | `.post-meta` (words ÷ ~220, rounded) |

Write the body with blocks copied from any existing post — `<p>`, `<h2>`, `<ul>`,
`<figure>`, and for code:

```html
<div class="codeblock">
  <div class="codebar"><span class="lang">go</span>
    <button class="copy" type="button"><svg …></svg><span>copy</span></button></div>
  <pre><code class="language-go">…</code></pre>
</div>
```

### 3. Wire up the rail

- Give every `<h2>`/`<h3>` an `id`, then list them in `<nav class="toc">`
  (`<a class="sub">` for the `h3`s).
- Point the **NEWER**/**OLDER** cards at the neighbouring posts — and open the
  post that used to be newest, whose NEWER card still says "you're at the
  latest", and point it back at this one.

### 4. List it on the home page

In `index.html`, the newest post is the `.featured` card. Demote the current one
into the first `.post-row` of the list, and put the new post in `.featured`.

### 5. List it in the archive

In `post/index.html`, add an `.arc-row` to the right `.year-group` (create the
group if it is a new year). **The `t-*` classes on the row drive the topic
filter** — a row tagged `go` needs `class="arc-row t-go"`, and the filter chip
ids in `.filters` must match (`#f-go` → `.t-go`). A brand-new topic also needs a
new chip and a new `:has()` pair in `css/page.css`.

### 6. Feeds, then ship

Add an `<item>` to `index.xml` and a `<url>` to `sitemap.xml`, then commit to
`master` and push — CI publishes to `gh-pages` within a minute.

## Unlisted posts

A post that exists at its URL but is not advertised anywhere — for sharing a
link with family or a few colleagues.

Build it exactly as above, then **skip steps 4 and 5, and the feed edits in
step 6**, and add one line to its `<head>`:

```html
<meta name="robots" content="noindex, nofollow">
```

That line is the **marker**: to find every unlisted post, grep for it.

```sh
grep -rl 'name="robots"' post/
```

So an unlisted post is:

- **not** in `index.html` (no `.featured` card, no `.post-row`)
- **not** in `post/index.html` (no `.arc-row`)
- **not** in `index.xml` or `sitemap.xml`
- carrying `noindex, nofollow`
- linked from nothing — including the NEWER/OLDER cards of its date-neighbours,
  which should skip straight over it

This is **unlisted, not private.** Anyone with the URL can read it, and the repo
is public, so the text is on GitHub either way. `noindex` asks search engines to
stay away; it is not access control. Do not put anything genuinely sensitive in
a post.

## Repo layout

```
index.html          home page — featured post + list (newest at the top)
post/
  index.html        archive page — year groups + CSS-only topic filter
  _template/        copy this to create a new post
  <slug>/index.html one directory per post
css/page.css        the whole design, one file
images/             post images
404.html            not-found page
index.xml           RSS feed (add a new <item> per post)
sitemap.xml         sitemap (add a new <url> per page)
CNAME               custom domain (blog.igormihalik.com)
.nojekyll           skip GitHub's Jekyll pass
```

## Design

App shell: a sticky **sidebar** (identity, nav), the **reading column** under a sticky **top bar** (breadcrumb + controls), and a **rail** whose contents change per page (year jump-list / table of contents). The rail drops below 1180px; below 860px the sidebar becomes a bottom nav.

- One CSS file (`css/page.css`): dark-first warm palette with a light theme, system font stacks (serif display / sans body / mono chrome), rounded code blocks with a language bar, native `<dialog>` lightbox for post images.
- Article width is one token, `--measure`. Heading, prose, code and figures all share it so they line up on a single left edge — widen or narrow the whole column from that one line.
- **The chrome makes no third-party requests** — no web fonts, no analytics; icons are inline SVG. (Some of the oldest posts still carry `gist.github.com` `<script>` embeds; those are the only outbound loads left, and inlining them as `.codeblock`s would remove them entirely.)
- Motion is CSS-only: scroll-linked reading progress (`animation-timeline: scroll()`), staggered post reveals (`view()`), and hover transitions. All of it collapses under `prefers-reduced-motion`.
- The archive topic filter is **radio inputs plus `:has()`** — filtering, including hiding emptied year groups, with zero JavaScript.
- Total JS: a theme toggle (`localStorage`) and copy-to-clipboard on code blocks, both inline, plus the lightbox on post pages.

## Deployment

Push to `master` → `.github/workflows/deploy.yml` force-pushes the tree to `gh-pages` → GitHub Pages serves it at [blog.igormihalik.com](https://blog.igormihalik.com). No build step, no third-party actions.

## Previewing

No tooling needed: open `index.html` in a browser straight from a checkout — internal links and assets are relative, so every page renders from the filesystem, from any local static server, and from GitHub Pages alike.
