# Field Notes

Concise, evidence-based notes on reliable AI agents, computer use, and personal infrastructure.

[![Deploy Hugo to GitHub Pages](https://github.com/Trac3r00/ai-autopilot-blog/actions/workflows/hugo.yaml/badge.svg)](https://github.com/Trac3r00/ai-autopilot-blog/actions/workflows/hugo.yaml)
[![Hugo 0.164.0](https://img.shields.io/badge/Hugo-0.164.0-ff4088?logo=hugo)](https://gohugo.io/)

## Overview

Field Notes is a static blog maintained by [Minseo Choi](https://github.com/Trac3r00). It is built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, then published to GitHub Pages by GitHub Actions.

The configured production URL is [trac3r00.github.io/ai-autopilot-blog](https://trac3r00.github.io/ai-autopilot-blog/).

## Features

- Markdown-based posts with TOML front matter generated from a content archetype
- Reading time, table of contents, breadcrumbs, post navigation, sharing links, and copy buttons for code blocks
- Automatic light and dark theme selection through PaperMod
- HTML, RSS, and JSON home-page output
- Minified production builds and automated GitHub Pages deployment from `main`

## Architecture

```text
content/posts/*.md ─┐
archetypes/         ├─> Hugo + hugo.toml ─> public/ ─> GitHub Pages
static/             │          ^
themes/PaperMod ────┘          └─ PaperMod Git submodule
```

Hugo combines the Markdown content, static files, site configuration, and PaperMod templates. The deployment workflow builds the generated `public/` directory and publishes it as a GitHub Pages artifact.

## Installation

### Prerequisites

- [Git](https://git-scm.com/)
- [Hugo Extended](https://gohugo.io/installation/) 0.164.0, matching the version used by CI

Clone the repository with its theme submodule:

```bash
git clone --recurse-submodules https://github.com/Trac3r00/ai-autopilot-blog.git
cd ai-autopilot-blog
```

If the repository was cloned without submodules, initialize the theme separately:

```bash
git submodule update --init --recursive
```

## Usage

Start the local development server, including draft content:

```bash
hugo server --buildDrafts
```

Hugo prints the local preview URL when the server starts and rebuilds the site as files change.

Create a post from the repository's default archetype:

```bash
hugo new content posts/my-post.md
```

New posts are created as drafts. Edit the generated file under `content/posts/`, then set `draft = false` when it is ready to publish.

Create a minified production build locally:

```bash
hugo --minify
```

The generated site is written to `public/`, which is ignored by Git.

## Configuration

Site configuration is stored in [`hugo.toml`](hugo.toml). It defines:

- the production base URL and site metadata;
- the PaperMod theme and presentation options;
- syntax highlighting and Markdown rendering;
- HTML, RSS, and JSON outputs; and
- asset minification settings.

There are no application environment variables. The deployment workflow sets `HUGO_VERSION=0.164.0` for its build job and overrides `baseURL` with the URL provided by GitHub Pages.

Files copied directly into the generated site live under `static/`. This currently includes `robots.txt` and the placeholder `ads.txt` file.

## Development

Before submitting documentation or content changes, run a production build:

```bash
hugo --minify
```

To reproduce the deployment build with an explicit base URL:

```bash
hugo --minify --baseURL "https://trac3r00.github.io/ai-autopilot-blog/"
```

Pushes to `main` trigger [the GitHub Pages workflow](.github/workflows/hugo.yaml). The workflow checks out submodules recursively, installs Hugo Extended 0.164.0, builds the site, and deploys the `public/` directory. It can also be started manually with `workflow_dispatch`.

## Project structure

```text
.
├── .github/workflows/hugo.yaml   # GitHub Pages build and deployment
├── archetypes/default.md         # Front matter for new content
├── content/posts/                # Published and draft posts
├── static/                       # Files copied directly to the site root
├── themes/PaperMod/              # PaperMod Git submodule
└── hugo.toml                     # Hugo and PaperMod configuration
```

## License

This repository does not currently include a license file. Unless a license is added, the project remains under its copyright holder's applicable rights.
