# Saptarshi Dutta's Portfolio — Hugo Site

This is the Hugo-migrated version of the original static HTML portfolio.

## Structure

```
hugo-site/
├── hugo.toml                  # Site config (baseURL, params)
├── archetypes/
│   └── default.md             # Template for new posts
├── content/
│   ├── _index.md              # Homepage intro text
│   └── blog/
│       ├── building-a-shell-from-scratch.md
│       ├── building-a-http-server-from-scratch.md
│       └── books-i-loved-during-undergrad.md
├── data/
│   ├── projects.yaml          # Project list (edit to add/remove projects)
│   └── books.yaml             # Book list (edit to add/remove books)
├── layouts/
│   ├── index.html             # Homepage template
│   └── _default/
│       ├── baseof.html        # Base HTML shell (shared by all pages)
│       ├── single.html        # Blog post template
│       ├── list.html          # Blog listing template
│       └── books.html         # Books page template
└── static/
    └── css/
        └── main.css           # All styles (retro theme)
```

## Getting Started

### Prerequisites
Install Hugo: https://gohugo.io/installation/

### Run locally
```bash
cd hugo-site
hugo server -D
```
Then open http://localhost:1313

### Build for production
```bash
hugo --minify
```
Output goes to the `public/` directory — deploy that folder anywhere (Netlify, GitHub Pages, Cloudflare Pages, etc.).

## Adding a New Blog Post
```bash
hugo new blog/my-new-post.md
```
Then edit `content/blog/my-new-post.md`.

## Adding a Project
Edit `data/projects.yaml` and add a new entry:
```yaml
- name: MyProject
  url: https://github.com/Saptarshi2001/MyProject
  lang: Go
  description: What it does
```

## Deploying to GitHub Pages
1. Push the repo to GitHub
2. In repo Settings → Pages → Source: GitHub Actions
3. Add `.github/workflows/hugo.yml` using the official Hugo workflow

## Customising
- Site-wide settings (name, email, social links): `hugo.toml` → `[params]`
- Colors and fonts: `static/css/main.css` → `:root` variables
