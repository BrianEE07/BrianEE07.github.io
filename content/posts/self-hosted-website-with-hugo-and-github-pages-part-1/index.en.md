---
title: "Hugo × GitHub Pages – Part 1: Running Your First Site Locally with Hugo"
date: 2025-12-27T21:22:40+08:00
draft: false
aliases:
  - "Hugo × GitHub Pages – Part 1: Running Your First Site Locally with Hugo"
description: This article walks through creating a local Hugo project, configuring a theme, and writing the first Markdown post to run a static website locally.
slug: self-hosted-website-with-hugo-and-github-pages-part-1
categories: hugo-self-hosted-website
tags:
  - hugo
  - github_pages
  - personal_website
---
> In the previous article, I talked about the basic ideas behind building a static website.
> 
> 👉 [Hugo × GitHub Pages – Part 0: Core Concepts and Website Building Blocks](/en/posts/self-hosted-website-with-hugo-and-github-pages-part-0/)
> 
> In this article, we will create a Hugo project, set up a theme, and build our very first Markdown-based webpage locally.

## 1. What Is Hugo?

![](assets/index/file-20251226161515782-572fa2ce-59be-4ca3-abf2-dcfb883f4fc6.png)

Hugo is a **static site generator (SSG)** that converts your written Markdown content into a complete website consisting of HTML, CSS, and JavaScript files that can be directly published online.

```
Markdown content + theme + configuration
            ↓
        Hugo builds
            ↓
A complete static website (HTML / CSS / JS)
```

On the [official Hugo website](https://gohugo.io/), Hugo describes itself with this line:

> *The world’s fastest framework for building websites*

Honestly, when I first saw this, my immediate reaction was:

**That’s a bold claim.**

But after actually using it, you’ll realize this isn’t just marketing language.

Hugo’s biggest strengths are that it is **fast, simple, and does not rely on a heavy frontend ecosystem**.

Hugo itself is a single executable and does not require Node.js to run. If your theme uses npm, Go modules, or a separate Sass tool, install those dependencies as its documentation requires. Once the content and configuration are ready, Hugo can quickly generate the site.

Compared to tools like [Hexo](https://hexo.io/zh-tw/) or [Jekyll](https://jekyllrb.com/), which require additional runtime environments, Hugo is much simpler to install and maintain, with very few dependency-related issues.

---

## 2. Creating a Local Website with Hugo

I’m using macOS in this guide. For Windows users, please refer to the [official documentation](https://gohugo.io/getting-started/quick-start/).

### (1) Prerequisites

We’ll use [Homebrew](https://brew.sh/) to install Hugo. If you’ve never used Homebrew before, install it first by opening Terminal and running:

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew update && brew upgrade && brew cleanup
```

Install [Git](https://git-scm.com/install/mac) and Go for version control and any modules used by the theme (if Git is new to you, the later steps may take more effort):

```shell
brew install git go
```

Install Hugo:

```shell
brew install hugo
```

Run `hugo version` to check the installation. The output below is what I originally saw, using an older version. For a fresh installation of Blowfish, check its [minimum requirement](https://blowfish.page/docs/installation/) first, currently Hugo 0.162.0. Do not use the old version shown below as your installation target:

```shell
hugo version
```

```shell
hugo v0.152.2+extended+withdeploy darwin/arm64 BuildDate=2025-10-24T15:31:49Z VendorInfo=brew
```

### (2) Create a Hugo Project

This series uses `Weisblog` for the local project folder and the GitHub repository we will create later. The site title is **Was Wei weiß**. You can choose your own name, but use it consistently in the paths that follow.

Create a new project using `hugo new site <project-name>`:

```shell
hugo new site Weisblog
cd Weisblog
```

You should now see the following directory structure:

```shell
Weisblog/
|--- archetypes/
|--- assets/
|--- content/   # Markdown pages and posts
|--- data/
|--- i18n/
|--- layouts/
|--- static/
|--- themes/    # Themes are stored here
|--- hugo.toml  # Main configuration file
```

We’ll go into more detail about these directories and advanced Hugo settings later. For now, just remember:

1. Themes live under `themes/`
2. Pages and posts (Markdown files) live under `content/`

#### Download a Hugo Theme

Before the site can render properly, we need to choose a theme. You can browse available themes on [Hugo Themes](https://themes.gohugo.io/). Popular choices include [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/), [Stack](https://themes.gohugo.io/themes/hugo-theme-stack/), and [Blowfish](https://themes.gohugo.io/themes/blowfish/).

I personally use Blowfish. You can preview it on their [Demo Page](https://blowfish.page/).

![](assets/index/file-20251226161515777-bc5e8603-4edb-4147-8d88-2115f9697f7c.png)

You can further customize the theme later to create your own visual style. If you’re not sure which one to choose, Blowfish is a solid starting point. I plan to write a dedicated article about its customization options in the future.

Below, I’ll use Blowfish as an example. The installation process is similar for most themes, but it’s always recommended to follow the theme’s official documentation. The [Blowfish Installation Guide](https://blowfish.page/docs/installation/) is particularly clear.

Inside the `Weisblog/` directory, run:

```shell
git init
git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish

# Use this command in the future to update the theme
# git submodule update --remote --merge
```

Next:

```shell
rm hugo.toml   # Remove Hugo’s default config file
cp -r themes/blowfish/config config
```

This creates a new `config/` directory. Open `config/_default/hugo.toml` and uncomment the following line (**not** the one under `themes/blowfish/config/_default/`):

```shell
theme = "blowfish"
```

Next, open the copied `config/_default/languages.en.toml` in your project and change its existing `title` to your site name. Check any other enabled language files as well so they do not keep the theme’s sample title.

```toml
title = "Was Wei weiß"
```

The local preview screenshots below show the Blowfish defaults; the heading will change with your settings.

#### Create Your First Post

Now let’s create our first post using `hugo new content <path>`:

```shell
hugo new content posts/my-first-post.md
```

The new file will appear under the `content/` directory.

Inside the file, you’ll see metadata wrapped in `+++`. We’ll explain this in more detail later. For now, let’s simply write some Markdown content (If you’ve never written Markdown before, you can ask ChatGPT to generate a sample, and there are also plenty of resources available online). For example:

```markdown
+++
date = '2025-12-20T17:21:59+08:00'
draft = true
title = 'My First Post'
+++
# Hello world

This is my first post.
```

Now let’s see what the site looks like.

### **(3) Local Preview**

At this stage, we can start building and previewing the site. Here are the three most important Hugo commands:

| **Command**      | **Purpose**       | **What It Does**                              | **Common Use Case**               |
| ---------------- | ----------------- | --------------------------------------------- | --------------------------------- |
| `hugo`           | Production build  | Generates the full static site into `public/` | Before deployment, GitHub Actions |
| `hugo server`    | Local development | Starts a local server with live reload        | Writing content, layout tweaking  |
| `hugo server -D` | Preview drafts    | Includes `draft = true` content               | Reviewing unpublished posts       |

Since our post is marked as `draft = true`, we need to run:

```shell
hugo server -D
```

`hugo server` provides the live preview. Running `hugo` produces the complete static website under `public/` for deployment in the next article.

Based on the output, open the local URL in your browser—usually `http://localhost:1313/`:

```shell
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

![](assets/index/file-20251226161515778-b405e5d6-9db7-49a5-86a8-9785a1a97d70.png)

Because our post is under `content/posts/`, navigating to `http://localhost:1313/posts/` will show the newly created article.

![](assets/index/file-20251226161515781-95a15c95-d187-433b-b10d-1e5d3922992a.png)

![](assets/index/file-20251226161515782-4537e462-87fc-445e-a32d-023b25492f96.png)

Try editing the post and saving the file—you’ll see the preview update **instantly**, which makes writing extremely convenient.

> In the next article, we’ll complete the **first real deployment**, automatically deploying the Hugo site to **GitHub Pages**.
> 
> 👉 [Hugo × GitHub Pages – Part 2: First Deployment with GitHub Actions](/en/posts/self-hosted-website-with-hugo-and-github-pages-part-2/)
