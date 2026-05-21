---
title: "Create Blog with Hugo and Github Pages"
date: 2026-04-02
draft: false
categories: ["Github", "Blogging"]
tags: ["github-pages", "hugo", "setup"]
---

1. Install Hugo from [Hugo Releases] (https://github.com/gohugoio/hugo/releases).

For Windows, Download: hugo\_\*extended\*\_windows-amd64.zip \
Extract it and add the folder to your PATH. Verify: ```bash hugo version ```

2. Create Your Hugo Site

In terminal enter  ```bash hugo new site crz-blog```. Go into it.

3. Initialize Git 

```bash init git ```

4. Add a Theme

Let's see how to install [PaperMod] (https://github.com/adityatelange/hugo-PaperMod) which is a good theme. \
```bash git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod ```

5. Configure Hugo

Open hugo.toml and replace contents with:

```yaml

baseURL = 'https://YOUR_GITHUB_USERNAME.github.io/'
languageCode = 'en-us'
title = 'YOUR_BLOG_TITLE'
theme = 'PaperMod'

[pagination]
pagerSize = 10

[params]
defaultTheme = "dark"
ShowCodeCopyButtons = true
ShowReadingTime = true
ShowBreadCrumbs = true
ShowPostNavLinks = true

[taxonomies]
category = "categories"
tag = "tags"

```
Replace: YOUR_GITHUB_USERNAME, and YOUR_BLOG_TITLE with actual values.

6. Create Your First Post

```bash hugo new posts/hello-world.md ```

Edit the generated file:

```text
---
title: "Hello World"
date: 2026-05-21
draft: false
categories: ["General"]
tags: ["intro"]
---

My first Hugo post.
```

7. Run Locally

```bash
hugo server
```
Open: http://localhost:1313. You now have a working blog.

8. Create GitHub Repository

Go to [GitHub] (https://github.com). \
Create repository: YOUR_GITHUB_USERNAME.github.io 

IMPORTANT: exact naming matters Example: christyjohn.github.io

9. Push Your Site

Add remote: 

```bash 
git remote add origin https://github.com/YOUR_USERNAME/YOUR_USERNAME.github.io.git 
```

Commit:
```bash 
git add .
git commit -m "Initial Hugo site"
git push -u origin main
```

10. Setup GitHub Actions Deployment

Create folder:
```bash 
.github/workflows
```
Create file: hugo.yml

Add:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Deploy
        id: deployment
        uses: actions/deploy-pages@v4
```

Commit and push:

```bash
git add .
git commit -m "Add GitHub Pages deployment"
git push
11. Enable GitHub Pages
```

In your repository:

Settings → Pages

Under:

Source → select “GitHub Actions”

Done.

After a minute or two, your blog will be live at:

https://YOUR_USERNAME.github.io


12. Recommended Content Structure

For your style of blogging:

content/
  category-1/
  category-2/
  category-3/

This will scale beautifully later.

13. Useful Hugo Commands

Create post:

``` 
hugo new ask-the-ai-bot/java-generics.md
```

Run locally:  
```
hugo server
```

Build site:
```
hugo
```