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