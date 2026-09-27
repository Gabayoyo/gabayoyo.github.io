# gabayoyo.github.io

Personal site for Aaron Gabayoyo, built with [Jekyll](https://jekyllrb.com/) and served by
GitHub Pages. Plain HTML, Sass and markdown — no build tooling beyond Jekyll itself.

Live at [gabayoyo.github.io](https://gabayoyo.github.io).

## Running locally

Requires Ruby 2.7 or newer. macOS ships 2.6, which is too old for Jekyll 4 — point `PATH` at a
newer Ruby if `bundle` fails.

```bash
bundle install
bundle exec jekyll serve
```

The site is served at `http://127.0.0.1:4000` and rebuilds on changes. **Editing `_config.yml`
requires a restart** — the config is read once at boot, so a running server will not pick it up.

## Structure

- `index.md` — the About page, served at `/`
- `blog.md`, `_posts/` — the blog index and posts
- `_includes/`, `_layouts/`, `_sass/` — shared markup, page templates and styling
- `files/` — static assets (the CV and project report), served at `/files/`
- `images/` — favicon and app icons

## Deploying

Push to `main`. Pages builds the site itself under **Settings → Pages → Deploy from a branch**,
so there is no workflow file and nothing to run before pushing.
