# igm.github.io

Source for [blog.igormihalik.com](https://blog.igormihalik.com) — a [Hugo](https://gohugo.io) site deployed to GitHub Pages.

## Setup

```bash
hugo server -D    # local dev server at http://localhost:1313
hugo              # production build into ./public
```

## Layout

- `content/post/` — blog posts (Markdown)
- `layouts/` — templates (`_default/baseof.html` is the site shell; partials in `layouts/partials/`)
- `static/` — CSS, images and other assets served as-is
- `.github/workflows/gh-pages.yml` — builds on push to `master` and publishes to the `gh-pages` branch

## Writing a post

Create `content/post/my-post.md`:

```markdown
+++
date = 2026-01-01T10:00:00+01:00
title = "My post"
+++

Intro paragraph — this appears on the home page list.

<!--more-->

Rest of the post.
```

## Deployment

Push to `master` → GitHub Actions builds with Hugo and deploys to `gh-pages` → served at [blog.igormihalik.com](https://blog.igormihalik.com) (CNAME).
