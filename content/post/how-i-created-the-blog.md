---
title: "Site Setup and How to Write Articles"
date: 2023-01-01
draft: false
summary: "How this site was built using Hugo and the GitHub-style theme..."
---

How this site was built using Hugo and the GitHub-style theme MeiK2333
<!--more-->

This site is set up using the [GitHub-style](https://github.com/MeiK2333/github-style) theme by MeiK2333, selected for its minimal aesthetic and GitHub-inspired layout. It's ideal for clear technical documentation and articles. The static site generator behind it is [Hugo](https://gohugo.io/), chosen for its speed and simplicity.

To build this site, a new Hugo project was created using:

```bash
hugo new site name
cd name
```

Inside that directory, Git was initialized:

```bash
git init
```

The theme was then added as a Git submodule:

```bash
git submodule add https://github.com/MeiK2333/github-style.git themes/github-style
```

Next, the `hugo.toml` file was configured with basic site information and the theme reference:

```toml
baseURL = "your url"
languageCode = "en-us"
title = "Kjersheim – Projects & Docs"
theme = "github-style"

[params]
  author = "name"
  description = "title"
  github = "name"
  avatar = "/images/avatar.png"
  headerIcon = "/images/avatar.png"

[frontmatter]
  lastmod = ["lastmod", ":fileModTime", ":default"]
```

Images like `avatar.png` are stored in the `static/images/` directory, so they're accessible as `/images/avatar.png`.

To create a new article, run:

```bash
hugo new post/my-article.md
```

Then open the generated file in `content/post/` and make sure the front matter looks like this:

```yaml
---
title: "My Article"
date: 2024-05-27
draft: false
summary: "A short summary for previews."
---
```

Be sure that the date is not in the future and that `draft` is set to `false`.

For controlling preview content on the homepage, use the `<!--more-->` tag:

```markdown
This content appears in the preview.
<!--more-->
This part only appears on the full post page.
```

Standard Markdown formatting works as expected:

```markdown
## Headings

**Bold**, *Italic*, `inline code`

- Lists
- Like this

> Blockquotes

```bash
Code blocks
```
```

To preview your site locally, run:

```bash
hugo server -D
```

and visit [http://localhost:1313](http://localhost:1313) in your browser.

For extra features like math support, add `katex: math` to the front matter and use `\\( inline \\)` or `$$ block $$` syntax.

Collapsible sections can be added like this:

```markdown
{{<details "Click to expand">}}

Hidden content goes here.

{{</details>}}
```

To deploy the site, you can use GitHub Pages and optionally set up a script using `gh-pages` and `hugo -t github-style`.

For full documentation, visit:

- [GitHub-style Theme](https://github.com/MeiK2333/github-style)
- [Hugo Docs](https://gohugo.io/documentation/)
