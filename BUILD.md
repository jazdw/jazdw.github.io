# Build and preview

This is a Jekyll website. Ruby runs Jekyll, and Bundler installs the Ruby
packages (gems) pinned in `Gemfile.lock`. The site targets the same versions
GitHub Pages builds: Jekyll 3.10.0 via the `github-pages` 232 gem.

## Prerequisites

- Homebrew
- Homebrew's `ruby@3.3` formula (`brew install ruby@3.3`)

macOS's built-in `/usr/bin/ruby` (2.6.x) is too old and is left untouched.
`ruby@3.3` is a "keg-only" formula, so Homebrew does not put it on your `PATH`
automatically.

## Use Ruby 3.3 in a terminal

Add Ruby 3.3 to `PATH` **for the current terminal session only**. This does not
modify your shell configuration and ends when you close the terminal:

```sh
export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
```

Confirm the session is using it:

```sh
ruby --version   # expect 3.3.x
bundle --version # expect 4.0.x
```

Ruby 3.3 ships the Bundler version recorded in `Gemfile.lock`, so there is
nothing extra to install. If you are in a new terminal, run the `export`
command again.

## Install dependencies

From the repository root:

```sh
bundle install
```

This downloads the pinned Jekyll and plugin gems. Repeat it only when
dependencies change.

## Build and preview

Build the static site into `_site/`:

```sh
bundle exec jekyll build
```

Run a local preview that rebuilds when source files change:

```sh
bundle exec jekyll serve --livereload
```

Then open <http://localhost:4000>. `bundle exec` uses this project's pinned
gems rather than other versions that might be installed. The generated `_site/`
directory is ignored by Git.

## Deployment

`.github/workflows/pages.yml` builds and deploys the site to GitHub Pages using
GitHub's official Jekyll Pages action. It runs only when manually started from
the repository's Actions tab.

In the repository settings, set **Pages → Build and deployment → Source** to
**GitHub Actions**.

The official builder runs GitHub Pages gem **232**, which pins Jekyll
**3.10.0**. Upstream Jekyll 4.x is newer but is not supported by the official
Pages builder.
