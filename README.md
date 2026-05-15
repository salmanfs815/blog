# My New Hugo Site

A blog built with [Hugo](https://gohugo.io/) using the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## Prerequisites

- [Hugo](https://gohugo.io/installation/) (v0.87.0 or later recommended)
- [Git](https://git-scm.com/)

## Getting Started

### Clone the repository
```bash
git clone <your-repo-url>
cd blog
```

### Install theme submodule (if not already initialized)
```bash
git submodule update --remote --merge
```

## Development

### Run locally
Start the development server with draft and future content enabled:
```bash
hugo server -D
```

The site will be available at `http://localhost:1313/`

### Build the site
Generate the static site for production:
```bash
hugo
```

Output will be in the `public/` directory.

### Build with drafts
Include draft posts in the build:
```bash
hugo -D
```

## Content

- **Create a new post:**
  ```bash
  hugo new content/posts/my-new-post.md
  ```

- **Edit content:** Posts are located in the `content/` directory

- **Configure site:** Edit `hugo.yaml` for site settings

## Themes

This site uses the PaperMod theme. For theme-specific customizations, see the `themes/PaperMod/` directory.

## Directory Structure

- `content/` - Page and post content in Markdown
- `layouts/` - Custom layout templates
- `static/` - Static files (images, CSS, JS)
- `assets/` - Hugo pipes assets
- `themes/PaperMod/` - Theme directory
- `public/` - Generated static site (build output)

## Deployment

After building with `hugo`, deploy the contents of the `public/` directory to your hosting provider.
