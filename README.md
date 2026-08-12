# Saptarshi Dutta — Portfolio

A Hugo portfolio for [Saptarshi Dutta](https://github.com/Saptarshi2001), built with the PaperMod theme. It highlights selected projects and long-form writing, and deploys to GitHub Pages through GitHub Actions.

## Featured projects

- [AsyncLoad](https://github.com/Saptarshi2001/AsyncLoad)
- [JBroker](https://github.com/Saptarshi2001/JBroker)
- [Radsort](https://github.com/Saptarshi2001/Radsort)

Project metadata lives in `data/projects.yaml`. Entries marked `featured: true` appear on both the homepage and at the top of the Projects page.

## Run locally

Hugo Extended 0.146.0 or newer is required.

```bash
git clone --recurse-submodules https://github.com/Saptarshi2001/personal-website.git
cd personal-website
hugo server
```

Open `http://localhost:1313`.

If the repository is already cloned without its theme submodule, run:

```bash
git submodule update --init --recursive
```

## Add a blog post

```bash
hugo new content blog/my-new-post.md
```

Each post should include a title, description, and tags in its front matter.

## Deploy to GitHub Pages

The workflow at `.github/workflows/hugo.yml` builds and deploys every push to `main`.

1. Push the repository to GitHub.
2. Open **Settings → Pages** in the repository.
3. Set **Source** to **GitHub Actions**.
4. Push to `main`, or run the workflow manually from the **Actions** tab.

The workflow checks out the PaperMod submodule, builds with Hugo Extended, uploads the generated `public/` directory, and deploys it to the `github-pages` environment.

## Main files

- `hugo.toml` — site metadata, navigation, and Hugo settings
- `data/projects.yaml` — featured and additional projects
- `content/blog/` — blog posts
- `layouts/home.html` — portfolio homepage
- `layouts/projects/list.html` — Projects page
- `assets/css/extended/portfolio.css` — portfolio styles
- `.github/workflows/hugo.yml` — GitHub Pages deployment
