# The Third Gate 待定之地

Personal blog built with [Hugo](https://gohugo.io/) and [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack/), deployed at [andornot.xyz](https://andornot.xyz).

## Setup

```bash
# Clone with submodules
git clone --recursive https://github.com/igaozp/igaozp.github.io.git

# Install dependencies
npm install --dev @babel/cli @babel/core @babel/preset-env browserslist clipboard cssnano postcss-cli postcss-import postcss-mixins postcss-nested postcss-preset-env postcss-url

# Start local dev server
hugo server
```

Requires Hugo extended version.

## New Post

```bash
# Single markdown file
hugo new posts/post-title.md

# Page bundle (with images)
hugo new posts/post-title/index.md
```

## Deployment

Pushing to `master` triggers GitHub Actions to build and deploy to the `gh-pages` branch automatically.
