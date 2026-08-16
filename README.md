# igm.github.io

Source for [blog.igormihalik.com](https://blog.igormihalik.com) — plain HTML, no static site generator. What's in `master` is exactly what gets served.

## Adding a post

1. `cp -r post/_template "post/my-post-slug"` — the directory name **is** the URL: `/post/my-post-slug/`
2. Edit the copy — replace `POST TITLE`, `YOUR-SLUG`, dates, and write the content. Copy content blocks (`<p>`, `<pre><code>`, `<img>`) from any existing post.
3. Add it to the top of the list in `index.html` (title + date + link) and in `post/index.html` (same).
4. Commit to `master` and push — CI publishes to `gh-pages` (the live branch) within a minute.

## Repo layout

```
index.html          home page — post list (new posts go at the top)
post/
  index.html        archive page
  _template/        copy this to create a new post
  <slug>/index.html one directory per post
css/page.css        the whole design, one file
images/             post images
404.html            not-found page
CNAME               custom domain (blog.igormihalik.com)
.nojekyll           skip GitHub's Jekyll pass
```

## Design

- One CSS file (`css/page.css`): warm paper palette, system fonts, ~44rem reading column, dark rounded code blocks, native `<dialog>` lightbox for post images.
- No JavaScript dependencies — the only JS is the inline lightbox and the Disqus embed, both inside each post page.
- Disqus threads are keyed by `this.page.identifier = '/post/<slug>/'` — keep that unchanged when editing a post so existing comments stay attached.

## Deployment

Push to `master` → `.github/workflows/deploy.yml` force-pushes the tree to `gh-pages` → GitHub Pages serves it at [blog.igormihalik.com](https://blog.igormihalik.com). No build step, no third-party actions.
