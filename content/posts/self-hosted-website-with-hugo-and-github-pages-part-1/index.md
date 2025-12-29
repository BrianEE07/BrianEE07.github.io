---
title: 手把手自架個人網站｜Hugo × GitHub Pages – Part 1：用 Hugo 在本地跑起第一個網站
date: 2025-12-27T21:22:34+08:00
draft: false
description: 本篇實作使用 Hugo 建立本地專案，設定主題並撰寫第一篇 Markdown 文章，實際跑起第一個靜態網站，作為 Hugo × GitHub Pages 系列的實作起點。
slug: self-hosted-website-with-hugo-and-github-pages-part-1
categories: hugo-self-hosted-website
tags:
  - Hugo
  - GitHub-Pages
  - Personal-Website
---
 > 上一篇我介紹了架設靜態網站的基本觀念。
 > 
 > 👉 [手把手自架個人網站｜Hugo × GitHub Pages – Part 0：先搞懂觀念與架站元素，再開始動手](/posts/self-hosted-website-with-hugo-and-github-pages-part-0/)
 > 
 > 本篇用 Hugo 建立專案、設定主題，完成本地上第一篇 Markdown 文章網頁。

## 一、Hugo 是什麼？

![](assets/index/file-20251226161515782-572fa2ce-59be-4ca3-abf2-dcfb883f4fc6.png)

Hugo 是一個**靜態網站產生器（Static Site Generator）**，用來把你寫好的 Markdown 內容，轉換成可以直接放上網的 HTML、CSS 與 JavaScript。

```
Markdown 文章 + 主題(theme) + 設定檔
            ↓
        Hugo 編譯
            ↓
    一整個靜態網站（HTML / CSS / JS）
```

在 [Hugo 官網](https://gohugo.io/)上，它給自己的介紹是這一句：

> *The world’s fastest framework for building websites*

老實說，第一次看到真的會忍不住想說一句：

**這麼囂張？**

但實際用過之後，會發現這句話並不是單純的行銷用語。

Hugo 最大的特色在於：**快、單純、而且不依賴龐大的前端生態系**。

它是一個單一執行檔的工具，不需要 Node.js、不需要安裝一堆套件，只要把內容與設定準備好，就能快速產生整個網站。

相較於 [Hexo](https://hexo.io/zh-tw/) 或 [Jekyll](https://jekyllrb.com/) 這類需要額外執行環境的工具，Hugo 在安裝與維護上更單純，也幾乎沒有套件相依問題。

---

## 二、用 Hugo 建立本地網站

筆者使用的是 MacOS，Windows 系統請參考 [官方教學](https://gohugo.io/getting-started/quick-start/)。

### (1) 前置作業

接下來會使用 [Homebrew](https://brew.sh/) 裝 Hugo，若第一次使用要安裝一下，開啟 Terminal：

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew update && brew upgrade && brew cleanup
```

若沒有 [Git](https://git-scm.com/install/mac) 也要裝一下（但如果沒用過 Git 之後可能會比較辛苦 QQ）：

```shell
brew install git
```

安裝 Hugo：

```shell
brew install hugo
```

輸入 `hugo version` 查看版本，看到版本資訊即成功安裝：

```shell
hugo version
```

```shell
hugo v0.152.2+extended+withdeploy darwin/arm64 BuildDate=2025-10-24T15:31:49Z VendorInfo=brew
```

### (2) 建立 Hugo 專案

輸入創建專案指令 `hugo new site <專案名稱>`：

```shell
hugo new site mysite
cd mysite
```

會發現資料夾結構如下：

```shell
mysite/
|--- archetypes/
|--- assets/
|--- content/  # 存放頁面、文章 Markdown 檔案
|--- data/
|--- i18n/
|--- layouts/
|--- static/
|--- themes/   # 存放主題
|--- hugo.toml # config 設定檔
```

之後有機會再介紹各個資料夾的用途和進階 Hugo 設定，現在要關注的就是：

1. 主題放在 `themes/` 底下
2. 待會新增的頁面（Markdown 檔案）會存放在 `content/` 底下

#### 下載 Hugo 主題

要先選擇一個主題，待會網頁才跑得出來。可以去 [Hugo Themes](https://themes.gohugo.io/) 找喜歡的，比較熱門的像是 [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/)、[Stack](https://themes.gohugo.io/themes/hugo-theme-stack/)、[Blowfish](https://themes.gohugo.io/themes/blowfish/) 都是不錯的選擇。

筆者用的是 Blowfish，可以點進去他們的 [Demo Page](https://blowfish.page/) 查看。

![](assets/index/file-20251226161515777-bc5e8603-4edb-4147-8d88-2115f9697f7c.png)

爾後也可以基於所選主題再自己微調、客制化，打造屬於自己的頁面風格。如果有選擇障礙不妨先試試 Blowfish，之後預期會有一篇介紹該主題的各種外觀設定。

以下用 Blowfish 為例，不同主題安裝教學大同小異，但仍建議去主題官方查看步驟，像是 [Blowfish Installation Guide](https://blowfish.page/docs/installation/) 就寫得蠻清楚。

在剛剛建立的 `mysite/` 底下：

```shell
git init
git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish

# 未來更新可用此指令
# git submodule update --remote --merge
```

接著，

```shell
rm hugo.toml   # 刪掉 Hugo 預設的，使用 Blowfish 的 config 設定檔
cp -r themes/blowfish/config config
```

會多一個複製過來的 `config/` 路徑，接著要初步編輯 `config/_default/hugo.toml`，將下面那一行取消註解。（注意**不是** `theme/blowfish/config/_default/` 底下的喔）

```shell
theme = "blowfish" # UNCOMMENT THIS LINE`
```

#### 新增文章

接著我們來新增第一篇文章，指令為 `hugo new content <檔案路徑>`。

```shell
hugo new content posts/my-first-post.md
```

新增的檔案會在 `content/` 底下。

可以看到檔案中會有一些用 `+++` 包起來的 metadata，這之後會有更詳細的教學。我們可以先試試開始寫 Markdown 語法（沒寫過也可以請 ChatGPT 生一段，網路上也很多資源）。舉例如下：

```markdown
+++
date = '2025-12-20T17:21:59+08:00'
draft = true
title = 'My First Post'
+++
# Hello world

This is my first post.
```

接著我們來看看網頁會長什麼樣子吧～

### (3) 本地端測試

這個階段開始編譯網站，簡單介紹以下三種主要指令：

| **指令**           | **主要用途** | **會發生什麼事**               | **常見使用時機**         |
| ---------------- | -------- | ------------------------ | ------------------ |
| `hugo`           | 正式編譯網站   | 產生完整靜態網站，輸出到 `public/`   | 部署前、GitHub Actions |
| `hugo server`    | 本地開發預覽   | 啟動本地伺服器，檔案變更即時更新         | 寫文章、調整版型           |
| `hugo server -D` | 預覽草稿     | 連 `draft = true` 的文章一起顯示 | 尚未發布的內容檢查          |

可以看到我們剛剛新增的文章有個 `draft = true`，代表這篇還是草稿，所以我們輸入：

```shell
hugo server -D
```

**產生的 `public/` 就是一整包靜態網站！**

依照輸出的提示，在瀏覽器上輸入 `localhost` 網址，通常是 `http://localhost:1313/`，就會看到很陽春的頁面：

```shell
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

![](assets/index/file-20251226161515778-b405e5d6-9db7-49a5-86a8-9785a1a97d70.png)

剛剛的文章是在 `content/posts/` 底下，所以我們瀏覽器輸入 `http://localhost:1313/posts/`，就可以看到我們剛剛新增的文章了！

![](assets/index/file-20251226161515781-95a15c95-d187-433b-b10d-1e5d3922992a.png)

![](assets/index/file-20251226161515782-4537e462-87fc-445e-a32d-023b25492f96.png)

另外可以嘗試編輯文章內容後儲存，網站也會**即時更新預覽**，很方便。

> 下一篇，會接著完成**第一次正式上線**，把 Hugo 網站自動部署到 **GitHub Pages**。
> 
> 👉 [手把手自架個人網站｜Hugo × GitHub Pages – Part 2：首次上線，透過 GitHub Actions 自動部署 Hugo 網站到 GitHub Pages](/posts/self-hosted-website-with-hugo-and-github-pages-part-2/)
