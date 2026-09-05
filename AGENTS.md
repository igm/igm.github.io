# AGENTS.md

Guidance for AI agents working in this repo.

**Read [README.md](README.md) first.** It is the source of truth for what this
site is, how to add a post, how off-index posts work, the repo layout, the
design system, and how deployment works. Everything below is the extra context
an agent needs that the README does not cover — the invariants that are easy to
break.
Do not restate README content here; if something changes, update the README and
leave this file pointing at it.

## The one thing to understand

There is **no static site generator and no build step**. Every `.html` file in
`master` is served verbatim. That has two consequences agents get wrong:

- **Never introduce a build step, bundler, framework, or `package.json`** to
  "make this easier". The absence of tooling is the point.
- **Every page carries the whole shell inline** — sidebar, top bar, rail and two
  inline scripts. A change to the shell means editing every page consistently.
  Scripting a bulk edit is fine, but the script is a throwaway: keep it out of
  the repo (use the scratchpad) and commit only the resulting HTML.
- **Because of that, the shell holds nothing that changes when a post is
  published** — no counts, no topic list, no dated note. The sidebar is
  byte-identical everywhere. Keep it that way: anything post-dependent belongs
  on `index.html` or `post/index.html`, which you are editing anyway.

## Invariants — do not break these

| Rule | Why |
|---|---|
| No third-party requests | No web fonts, CDNs, or analytics. Icons are inline SVG; fonts are system stacks. (Some of the oldest posts still have `gist.github.com` embeds — pre-existing, see README.) |
| Keep the JS budget tiny | Theme toggle + clipboard + lightbox, all inline. No frameworks. The archive topic filter is **CSS-only** (`:has()` + radio inputs) — do not "fix" it with JavaScript. |
| Relative links, always ending in a file | Link to `index.html`, never a bare directory (`href="../../"`), or local `file://` preview opens a directory listing. |
| `--measure` is the single article width | Heading, prose, code and figures share it so they align on one left edge. Change the width there, not per-element. |
| Nothing post-dependent in the shell | No counts, no topic list, no "currently" note — deliberately, so publishing never means editing every page. Do not "helpfully" reintroduce a post count, a topic tally, stat tiles, or a sidebar tag cloud. |

## Traps specific to this repo

- **Some posts are deliberately off-index** — `draft` or `unlisted`. Such a
  post is missing from `index.html`, `post/index.html`, `index.xml` and
  `sitemap.xml`, and its date-neighbours' NEWER/OLDER cards skip straight over
  it. That looks like a bug or a forgotten entry. It is not. Before adding any
  post to a listing or a count, check whether it is off-index:

  ```sh
  for f in post/*/index.html; do
    case "$f" in post/_template/*|post/hidden/*) continue;; esac
    s=$(sed -n 's/.*name="post-status" content="\([a-z]*\)".*/\1/p' "$f")
    [ -n "$s" ] && printf '%-9s /%s/\n' "$s" "$(dirname "$f")"
  done
  ```

  The `post-status` meta is the marker, so the answer is always in the files
  rather than in this document. See *Off-index posts* in the README.
- **`post/hidden/index.html` is a fourth listing to keep in sync** — and the
  only one an off-index post *should* appear in. It is linked from nothing by
  design, so it looks like an orphan; an agent tidying up "a page linked from
  nowhere" would delete exactly the wrong file. Don't.
- **`post/_template/index.html` carries the marker lines inside a comment**,
  and grep cannot tell comments from code — it must never be counted as an
  off-index post. The loop above skips it (and `post/hidden/`, the index
  itself) for exactly that reason.
- **Posts are not structurally uniform.** Some carry their own `<style>` and
  `<script>` blocks and bespoke classes with full-bleed layouts, and their
  scripts may live *outside* `<article>`. Bulk transforms that assume the
  standard post shape will silently mangle them — dropping a post's own scripts
  can leave its content stuck at `opacity: 0`, which is invisible rather than
  obviously broken. Where a post defines its own palette, keep it bound to the
  site theme tokens so it still works in both light and dark.
- Post body content lives in `<article class="post">`. Existing posts contain
  Word-export markup and legacy `<pre>` blocks; preserve them.

## Working style

- **Verify before claiming done.** Open the page and check it. `python3 -m
  http.server` in the repo root, or open the file directly — both must work.
  Check both light and dark themes, and a narrow viewport (the sidebar becomes a
  bottom nav below 860px).
- When editing post content, confirm you have not lost prose — compare word
  counts against `git show HEAD:<file>` before and after a bulk change.
- Do not commit or push unless asked.

## Related files

- [README.md](README.md) — the source of truth. Start there.
- `design.pen` — the Pencil design source for the current look. Edit it only
  with the `pencil` MCP tools; it is encrypted, so `Read`/`Grep` will not work.
