# Laurence Sadler portfolio

Personal portfolio and blog for Laurence Sadler, built with [Hugo](https://gohugo.io/) and [PaperMod](https://github.com/adityatelange/hugo-PaperMod).

Most of the site is Markdown. The home page, project cards, and a few layout overrides are custom. It deploys to GitHub Pages through GitHub Actions.

## Stack

- Hugo extended `0.126.0` (the version used by CI)
- PaperMod as a Git submodule at `themes/PaperMod`
- Custom layouts and CSS in `layouts/` and `assets/css/`
- Self-hosted Geist and Geist Mono fonts
- GitHub Pages deployment via `.github/workflows/hugo.yaml`

## Requirements

- Git
- Hugo extended `0.126.0` or a compatible newer version
- The PaperMod submodule

Node.js is not required for the current build. The Pages workflow installs Dart Sass and checks for npm lockfiles, but the site currently uses plain CSS.

## Local development

Clone with submodules:

```bash
git clone --recurse-submodules https://github.com/SirLorrence/SirLorrence.github.io.git
cd SirLorrence.github.io
```

If the repo is already cloned:

```bash
git submodule update --init --recursive
```

Start the development server:

```bash
hugo server -D
```

Open <http://localhost:1313/>. `-D` includes draft content.

Check the Hugo version:

```bash
hugo version
```

PaperMod requires the extended build.

## Production build

```bash
hugo --gc --minify
```

Hugo writes the output to `public/`, which Git ignores. To mirror CI more closely:

```bash
HUGO_ENVIRONMENT=production HUGO_ENV=production hugo --gc --minify
```

## Deployment

Pushes to `main` trigger `.github/workflows/hugo.yaml`, which:

1. Installs Hugo extended `0.126.0` and Dart Sass.
2. Checks out the repo with submodules.
3. Builds the site for GitHub Pages.
4. Uploads `public/` and deploys it.

Set the GitHub Pages source to GitHub Actions in the repository settings. The workflow overrides `baseURL` with the Pages URL, so local builds use `https://laurencesadler.com/` from `hugo.yaml`.

Do not commit `public/` or `resources/`; both are generated and ignored.

## Content

### Posts

Add posts under `content/posts/`:

```yaml
---
title: "Post title"
summary: "One-line summary"
author: Laurence Sadler
date: 2025-01-01T00:00:00+00:00
ShowReadingTime: true
tags: ["Hugo", "Tooling"]
---
```

### Projects

Projects live under `content/projects/`. They appear on the home page and the project index.

```yaml
---
title: "Project name"
summary: "Short card description"
tags: ["C#", "Unity"]
weight: 1
ShowToc: true
cover:
  image: "images/covers/project.png"
---
```

Useful project front matter:

- `weight`: controls ordering.
- `grid: false`: keeps the project out of the card grid and adds it as an "Also" link instead.
- `cover.image`: local path under `static/`, or an absolute URL. Project cards expect a cover; without one, the card renders as text only.
- `summary`: used for the card and metadata.
- `tags`: shown on the card and used by the tag pages.

Project cards convert GitHub `blob` cover URLs to `raw.githubusercontent.com`.

### Home page

Edit the home page's Experience, About, and Contact sections in `layouts/index.html`.

The old `/about/` URL redirects to `/#about`; there is no separate About content file to maintain.

## Layout and styling

```
layouts/
  index.html                 Custom home page
  _default/list.html         Lists, project index, blog index
  _default/single.html       Post/project detail pages
  _default/rss.xml           Custom RSS output
  partials/
    header.html              Site navigation
    footer.html              Site footer
    project-card.html        Home/project cards
    post_meta.html           Post metadata
    tags.html                Tag links
    extend_head.html         CSS, fonts, manifest

assets/css/
  main.css                   All site styles
  extended/blank.css         PaperMod extended-CSS placeholder
```

Notes:

- Do not edit `themes/PaperMod` directly. Put overrides in `layouts/` or `assets/`.
- The site uses PaperMod's head partial directly. Put project-specific `<head>` additions in `layouts/partials/extend_head.html`.
- `assets/css/main.css` is bundled and fingerprinted by `extend_head.html`.
- The hero image lives at `static/images/spiral-field-paper.png`. Update the `<img>` in `layouts/index.html` if it changes.
- Favicons and the web manifest live at the root of `static/`:
  - `favicon.svg`
  - `favicon-16.png`
  - `favicon-32.png`
  - `apple-touch-icon.png`
  - `icon-192.png` / `icon-512.png`
  - `site.webmanifest`
- Social preview images use `params.images` in `hugo.yaml` (`/images/Me.jpg`).
- The CV PDF lives at `static/laurence_sadler-cv.pdf`.

## Configuration

`hugo.yaml` holds the site title, base URL, social links, social preview image, and favicon/theme metadata. The custom header and footer links are hardcoded in their partials; there is no `menu` configuration to maintain.

## Reference links

### Hugo

- [Hugo documentation](https://gohugo.io/documentation/)
- [Hugo shortcodes](https://gohugo.io/content-management/shortcodes/)
- [Hugo front matter](https://gohugo.io/content-management/front-matter/)
- [Hugo image processing](https://gohugo.io/content-management/image-processing/)

### PaperMod

- [PaperMod demo/documentation](https://adityatelange.github.io/hugo-PaperMod/)
- [PaperMod wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [PaperMod FAQ: bundling custom CSS](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#bundling-custom-css-with-themes-assets)
- [PaperMod discussions](https://github.com/adityatelange/hugo-PaperMod/discussions/610)

### GitHub Pages

- [Using custom workflows with GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages)
- [actions/configure-pages](https://github.com/actions/configure-pages)
- [actions/deploy-pages](https://github.com/actions/deploy-pages)

### Design/fonts

- [Geist font](https://vercel.com/font)
- [Geist on GitHub](https://github.com/vercel/geist-font)
- [jessewei.dev: PaperMod notes](https://jessewei.dev/blog/2023/papermod/)

### Portfolio references

- [Cheerful Mousse portfolio](https://main--cheerful-mousse-b9d87b.netlify.app/)
- [Pascal Michaillet](https://pascalmichaillat.org/)
- [Pascal Michaillet: academic job market](https://pmichaillat.substack.com/p/going-on-the-academic-job-market)
