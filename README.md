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
