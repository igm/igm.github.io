# igm.github.io

Source for [blog.igormihalik.com](https://blog.igormihalik.com) — plain HTML, no static site generator. What's in `master` is exactly what gets served.

## Adding a post

There is no generator, so a new post is a copy plus a handful of deliberate edits.
Steps 1–3 create the post; steps 4–6 are what makes it *appear* on the site;
step 7 is the exit ramp for a post that should not appear yet.

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

### 7. Or keep it off-index

Steps 4–6 are what makes a post *appear*. If this one is a draft or unlisted,
skip steps 4 and 5 and the feed edits in step 6, and follow
[*Off-index posts*](#off-index-posts) instead: two marker lines in the head, a
banner under the header, and a row in `post/hidden/index.html`.

## Off-index posts

A post can exist at its URL without being advertised anywhere — a **draft**
you are still writing, or an **unlisted** post you are sharing by link with
family or a few colleagues. The site therefore has three states:

| State | `index.html` | `post/index.html` | Feeds | NEWER/OLDER | `post/hidden/` |
|---|---|---|---|---|---|
| **listed** — the default | ✅ | ✅ | ✅ | ✅ in the chain | — |
| **draft** | ❌ | ❌ | ❌ | ❌ skipped over | ✅ *Drafts* |
| **unlisted** | ❌ | ❌ | ❌ | ❌ skipped over | ✅ *Unlisted* |

Structurally, draft and unlisted are identical — same exclusions, same
`noindex`. The difference is intent: a draft has not been sent to anyone and
expects more editing; an unlisted post is finished and circulating. Keeping
them identical means one code path, and moving between them is a one-word edit.

Build the post exactly as steps 1–3 above, then instead of steps 4–6:

- add **both** marker lines to its `<head>`, immediately after `<title>`:

  ```html
  <meta name="robots" content="noindex, nofollow">
  <meta name="post-status" content="draft">      <!-- or: unlisted -->
  ```

- add the status banner as the first child of `<header class="post-header">`,
  above the `<h1>`:

  ```html
  <p class="statusnote"><b>unlisted</b>
    <span>Not listed on the site or in the feeds — but anyone with this link
    can read it.</span></p>
  ```

  For a draft: `<b>draft</b>` and "Unfinished and not shared yet."

- add a row to the matching section of `post/hidden/index.html` — the one page
  that lists off-index posts, so the only URL you have to remember is
  `/post/hidden/`.

`post-status` is the canonical marker: it is what the hidden index is
reconstructed from. To list every off-index post with its state:

```sh
for f in post/*/index.html; do
  case "$f" in post/_template/*|post/hidden/*) continue;; esac
  s=$(sed -n 's/.*name="post-status" content="\([a-z]*\)".*/\1/p' "$f")
  [ -n "$s" ] && printf '%-9s /%s/\n' "$s" "$(dirname "$f")"
done
```

(`_template/` carries the marker lines inside a comment and `hidden/` is the
index itself, so both are skipped; every other match is a real off-index post.
The two markers are always applied together, so `grep -rl 'name="robots"' post/`
still works as a coarser check.)

So an off-index post is:

- **not** in `index.html` (no `.featured` card, no `.post-row`)
- **not** in `post/index.html` (no `.arc-row`)
- **not** in `index.xml` or `sitemap.xml`
- carrying `noindex, nofollow` plus `post-status`
- wearing the `.statusnote` banner
- absent from the NEWER/OLDER chain — it has no `.updown` block of its own,
  and its date-neighbours' cards skip straight over it

This is **unlisted, not private.** Anyone with the URL can read it, and the repo
is public, so the text is on GitHub either way. `noindex` asks search engines to
stay away; it is not access control. Do not put anything genuinely sensitive in
a post. The hidden index makes *your* way back in easier; it does not make
anyone else's harder.

### Promoting an off-index post

**draft → unlisted — one word, plus two cosmetic follow-ups:**

```diff
-<meta name="post-status" content="draft">
+<meta name="post-status" content="unlisted">
```

Then change the banner wording, and move the row from the Drafts section to
the Unlisted section of `post/hidden/index.html` (swapping `st-draft` for
`st-unlisted`). Same URL, still `noindex`, still in no listing — you have just
started sharing the link.

**draft or unlisted → listed — six edits, then rejoin the chain.** None of the
six is optional: a post that misses one is half-listed, which looks fine on the
page you are checking and broken from every other one.

| # | File | Change |
|---|---|---|
| 1 | the post's `<head>` | delete **both** metas — `robots` and `post-status` |
| 2 | the post's `<header class="post-header">` | delete the `.statusnote` banner |
| 3 | `post/hidden/index.html` | delete its `.arc-row`; restore the `.empty-note` if that section is now empty |
| 4 | `index.html` | step 4 above — demote the current `.featured` into the first `.post-row`, promote this post |
| 5 | `post/index.html` | step 5 above — add an `.arc-row` to the right `.year-group` with matching `t-*` classes |
| 6 | `index.xml`, `sitemap.xml` | step 6 above — add an `<item>` and a `<url>` |

**Rejoining the NEWER/OLDER chain** has no counterpart in the publish flow, and
it is *not* "repoint two links": an off-index post has no `.updown` block at all
(neither `france-roadtrip-2026` nor `_template` has one — it is fully absent
from the chain, not sitting in it unlinked). Promotion means:

- copy an `.updown` block from a sibling post (e.g. the NEXT section of
  `post/coffee-driven-development/index.html`) into the promoted post, and
  point both of its cards at the correct date-neighbours;
- edit the **older** neighbour's NEWER card to point back at the promoted post;
- if the promoted post is now the newest, the previously-newest post's NEWER
  card still reads `— you're at the latest` and must become a real link.

Two traps, both of which fail silently:

- **A new topic needs three edits, not one** — a chip in `.filters`, the `t-*`
  class on the row, **and** a new `:has()` pair in `css/page.css`. Miss the
  third and the filter simply stops hiding that row; the page still renders.
  (Step 5 says this for new posts; it is repeated here because a promoted post
  is the likeliest source of a genuinely new topic.)
- **Same-day posts force a tie-break.** `france-roadtrip-2026` and
  `coffee-driven-development` are both dated 2026-08-16, so promoting the
  former means deciding by hand which is "newest" — and using that same answer
  in `index.html`, in the archive row order, and in the chain. There is no rule
  in the files to fall back on.

Demoting a listed post — back to unlisted or draft — is the same table read
upwards, plus the two marker lines and the banner. It is expected to be rare,
but the marker plus the banner make it a legitimate operation rather than a
hack.

## Repo layout

```
index.html          home page — featured post + list (newest at the top)
post/
  index.html        archive page — year groups + CSS-only topic filter
  hidden/           off-index posts — drafts + unlisted (linked from nothing)
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
