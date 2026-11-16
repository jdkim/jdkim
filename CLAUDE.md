# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Jin-Dong Kim's personal homepage: a single-page Jekyll site (kramdown/GFM) published on GitHub Pages at `https://jdkim.github.io/jdkim/` (`baseurl: "/jdkim"`). Remote: `git@github.com:jdkim/jdkim.git` (public). It previously lived at `http://dbcls.rois.ac.jp/~jdkim`.

## Build

```bash
bundle install
bundle exec jekyll build     # writes _site/
bundle exec jekyll serve     # local preview; restart after editing _config.yml
```

- `Gemfile` pins `ruby "3.4.12"` (RVM; the RVM default is 3.4.4, so `rvm use 3.4.12` first or `bundle` refuses to run). Jekyll is 4.4.x, which is required on Ruby 3.4 (4.3 lacks the `csv`/`base64` deps that left Ruby's default gems).
- Run Jekyll **from the repo root**. The custom `markdown` tag reads `File.join(Dir.pwd, "_includes", …)`, not Jekyll's source dir.

## Deploy

Pushing to `main` deploys: `.github/workflows/pages.yml` builds with Bundler (Ruby version from the `Gemfile`) and publishes via GitHub Pages' Actions source. GitHub's built-in Pages build is not used because it would refuse the custom `_plugins/`. Check runs with `gh run list --workflow pages.yml`.

`_site/` is still tracked in git from the pre-Pages days, but the deploy ignores it and builds fresh.

There are no tests or linters.

## How the page is put together

- `index.md` (layout `default`) is the whole site. It assembles sections from `_includes/*.md` with `{% markdown <file>.md %}`.
- `_plugins/markdown.rb` defines that `markdown` tag: it renders the include through Liquid (with the site payload), then through Kramdown on its own. This lets Markdown includes sit inside HTML `<div>`s in `index.md`, which plain `{% include %}` would not render. Kramdown is called directly there, so `_config.yml`'s `kramdown:` options (GFM input etc.) do **not** apply to these includes.
- `_plugins/font_awesome.rb` defines `{% icon fa-name [extra-class] %}` → `<i class="fa …">`. Font Awesome 4 is loaded from a kit script in `_includes/head.html`.
- `_layouts/default.html` + `_includes/head.html` + `css/main.css` hold all the layout and styling. `header.html`/`footer.html` and the `page`/`post` layouts are unused leftovers of the Jekyll scaffold.

## Editing content

Content updates almost always mean editing one of the `_includes/*.md` files:

- `activities.md` (upcoming + last 5 years, using the PC/SC/EC/OC abbreviations defined at its top) and `activities-recent.md` / `activities-past.md`
- `events.md`, `events-recent.md`, `events-past.md`: definition-list style (`Date, Place` then `: description`). The events blocks are currently hidden in `index.md` inside `<!-- -->`. Liquid still runs there, so they are rendered into the HTML comment, not skipped.
- `publications.md`, `projects.md`, `name.md`, `affiliation.md`, `contacts.md`, `education.md`
- The "Memberships" and "Research Interests" sections are written inline in `index.md`.

Files named `*-past.md`, `*-back.md`, `affiliation2.md`, `education.md` are not referenced from `index.md`; they are archives for moving old entries to.

Asset links use `{{ "/img/picture.jpg" | relative_url }}`, so they follow `baseurl`. Non-site files at the repo root (like `CLAUDE.md`) must be listed under `exclude:` in `_config.yml`, or Jekyll publishes them.
