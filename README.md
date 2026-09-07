# lhphanto.github.io

Personal site — [Jekyll](https://jekyllrb.com/) + [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/),
served by GitHub Pages from the `main` branch.

## Adding a post

Create `_posts/YYYY-MM-DD-slug.md`:

```markdown
---
title: "Post title"
date: 2026-09-07
categories:
  - notes
tags:
  - robotics
---

Body text here.
```

Commit and push — GitHub rebuilds in about a minute.

## Adding a page

Create `_pages/name.md` with a `permalink`, then add it to `_data/navigation.yml`
if it should appear in the top nav.

## Local preview (optional)

Requires Ruby 3.x — the macOS system Ruby (2.6) is too old.

```sh
brew install ruby
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc && exec zsh
bundle install
bundle exec jekyll serve --livereload
```

Then open http://localhost:4000

## Changing the look

Skin is set by `minimal_mistakes_skin` in `_config.yml`. Options: `air`, `aqua`,
`catppuccin_latte`, `catppuccin_mocha`, `contrast`, `dark`, `default`, `dirt`,
`mint`, `neon`, `plum`, `sunrise`.

Theme docs: https://mmistakes.github.io/minimal-mistakes/docs/configuration/

## Note on the Gemfile

The `github-pages` gem pins Jekyll 3.9, which cannot run on Ruby 3.2+ (it calls
`String#tainted?`, removed in Ruby 3.2). This Gemfile uses Jekyll 4 for local preview
instead, while GitHub Pages still builds the live site with its own Jekyll 3.10.

Minimal Mistakes supports both, so this is fine in practice. To make local and
production identical, switch the repo to a GitHub Actions build with Jekyll 4.

## Post series (build logs)

Posts in the `xlerobot` category are automatically collected into a series at
`/xlerobot/`, listed oldest-first.

To add a part:

```markdown
---
title: "Part 2: Assembly"
excerpt: "One-line summary shown on the series index."
date: 2026-09-14
categories:
  - xlerobot
tags:
  - build-log
toc: true
---

{% raw %}{% include series-nav.html %}{% endraw %}

Body text.
```

`_includes/series-nav.html` works out the post's own position and renders the
"Part N of M" box with prev/next links. Nothing to maintain by hand — ordering comes
from the post date.

**Gotcha:** Jekyll silently skips posts dated in the future. If a new post doesn't
appear, check its date against your current local time (the site timezone is
`America/Los_Angeles`).

To start a second series, copy `_pages/xlerobot.md` and `_includes/series-nav.html`,
swapping `site.categories.xlerobot` for your new category.
