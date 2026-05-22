---
title: "Enabling Comments in Hugo using Giscus"
date: 2026-04-21
draft: false
comments: true
categories: ["Github", "Blogging"]
tags: ["github-pages", "hugo", "setup"]
---

Let's see how we can enable comments in your blog using [Giscus](https://giscus.app/).

1. In [PaperMod](https://github.com/adityatelange/hugo-PaperMod)-my current themes, comments are disabled by default.

Enable them by adding this to your ```hugo.yaml / config.yaml```

```yaml
params:
  comments: true
 ```

2. Add this inside ```layouts/partials/comments.html```. 

*Note*: Must be exactly ```layouts/partials/comments.html```. 

Not inside, 
- ```layout/partials```
- ```themes/PaperMod/layouts/partials```
- ```comments.htm```

Add this inside ```layouts/partials/comments.html```

```javascript
<script
  src="https://giscus.app/client.js"
  data-repo="YOUR_USERNAME/YOUR_REPO"
  data-repo-id="YOUR_REPO_ID"
  data-category="Announcements"
  data-category-id="YOUR_CATEGORY_ID"
  data-mapping="pathname"
  data-strict="0"
  data-reactions-enabled="1"
  data-emit-metadata="0"
  data-input-position="bottom"
  data-theme="preferred_color_scheme"
  data-lang="en"
  crossorigin="anonymous"
  async>
</script>
```

You will get ```data-repo``` and ```data-repo-id``` from [Giscus](https://giscus.app/) app, while you are setting it up. 

*Note* - While setting it in Github Discussions, I selected 'General' as the category for the time being as my blog isin the initial stages.

3. Now, fix the global config.

In ```hugo.yaml``` add

```yaml
params:
  comments: true
 ```

4. Add to each post frontmatter

PaperMod can disable comments per post.

Inside a post:

```yaml
---
title: "My Post"
date: 2026-05-21
comments: true
---
```

5. Now restart you server and comments are enabled in your posts.

```hugo server --disableFastRender``` - the flag enables hugo server to refresh the caches.