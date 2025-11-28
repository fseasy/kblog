## Themes

```bash
cd themes
git clone https://github.com/CaiJimmy/hugo-theme-stack.git stack
```

## Write Flow



```bash
hugo new posts/25-1118-xxxx.md
# 1. you can organize by sub-dir. for each sub-dir, you can create a _index.md to make a entry-page or disable it.
# 2. use `/archetypes` to control the initial front matter. say, posts.md => new posts in `/content/posts`
# 3. Front Matter Usage
     1. image: head image for this doc (SEO / Show)
     2. math: true/false to enable/disable math render
     3. slug: permalink for the page
```