---
title: "Hugo × GitHub Pages – Part 0: Core Concepts and Website Building Blocks"
date: 2025-12-27T16:48:54+08:00
draft: false
description: Learn why a self-hosted personal website is worth building, understand static website fundamentals, and explore the core concepts behind using Hugo with GitHub Pages.
slug: self-hosted-website-with-hugo-and-github-pages-part-0
categories: hugo-self-hosted-website
tags:
  - Hugo
  - GitHub-Pages
  - Personal-Website
---
> The very first post on my personal website goes to a “how to build a website” guide 🙂

## 1. Why Build a Personal Website Yourself?

When it comes to writing online, the first platforms that usually come to mind are ready-made publishing platforms such as **Medium, Vocus, and Matters**.

These platforms have very clear advantages:

They take care of layout, hosting, and even basic SEO configuration for you. As a creator, you can focus almost entirely on writing content, with very little technical barrier to entry.

However, the drawbacks are just as clear: **you are completely dependent on the platform**.

Take Medium as an example. In recent years, many creators have noticed that article SEO performance has weakened, with decreasing visibility on Google search results. I personally experienced this as well—**even when searching with relevant keywords, I couldn’t find articles I had written myself**. Your content exists on the platform, but it doesn’t necessarily *belong* to you, nor is it guaranteed to be properly indexed or surfaced by search engines.

As I started thinking about long-term writing, one thing became increasingly clear:

**What I want is a website that truly belongs to me, not just an account under someone else’s platform.**

That’s why I decided to invest some upfront time into learning how to build my own website. While it does take extra effort at the beginning to understand the tools and workflow, once the structure is in place, I can return to focusing purely on writing. More importantly, the content, URLs, and presentation are all fully under my own control.

---

## 2. Most People Only Need a “Static Website”

When people hear “building your own website,” their first reaction is often:

Do I need to run a server?  
Do I need to write backend code?  
Isn’t that complicated?

In reality, **most personal websites don’t need to be dynamic at all**.

Simply put:

| Category | Dynamic Websites | Static Websites |
|------|------------------|-----------------|
| Request handling | Each request requires backend processing | Content is pre-generated (HTML / CSS / JS) |
| System requirements | Databases, servers, user systems | Browser directly loads files |
| Performance & maintenance | More complex, higher maintenance cost | Fast, stable, secure, low cost |
| Typical use cases | E-commerce, social platforms, online systems | Blogs, personal sites, portfolios |
| Target users | Services requiring real-time interaction | Content-focused creators |

For a personal website centered around written content, static websites are not only sufficient—they are often the better choice. They are easier to deploy, faster, more stable, and cheaper to maintain.

**What I want to build is not a complex system, but a clean, long-term maintainable static website.**

---

## 3. Website Building Can Be Reduced to Three Layers (Core Concept)

Before diving in, I spent some time clarifying the role of each tool. In the end, I broke the entire website-building process into three layers:

### The Three-Layer Website Model

- **Content Layer: Markdown**
  - Responsible for the content itself. Platform-independent and the most valuable long-term asset.
- **Generation Layer: Hugo**
  - Converts Markdown into a real website (HTML / CSS / JS) and manages site structure and styling.
- **Deployment Layer: GitHub Pages**
  - Hosts the generated static site online and provides HTTPS and domain access.

**Why Markdown?**

Markdown is one of the **most platform-independent content formats available**. My own note-taking system in Obsidian is also built entirely around Markdown.

**Why Hugo?**

There are other tools similar to Hugo, such as Jekyll and Hexo. Hugo stands out because:

- Its configuration and directory structure are relatively intuitive
- It does not require a JavaScript-heavy ecosystem
- **It is extremely fast**, making it ideal for frequent writing and previewing

**Why GitHub Pages?**

- More than sufficient for static websites
- Free, stable, and HTTPS-enabled
- Naturally integrates with Git-based version control workflows

Without the need for a backend or complex functionality, GitHub Pages is a **low-risk, low-cost** choice.

Once the roles of each layer and tool are clear, it’s time to start building. The following articles in this series will walk through the entire process step by step, turning this structure into a working website.

> In the next post, we’ll start from the very basics and run our first Markdown-powered site locally using Hugo.
> 
> 👉 [Hugo × GitHub Pages – Part 1: Running Your First Site Locally with Hugo](/en/posts/self-hosted-website-with-hugo-and-github-pages-part-1/)