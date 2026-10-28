---
title: 手把手自架個人網站｜Hugo × GitHub Pages – Part 2：首次上線，透過 GitHub Actions 自動部署 Hugo 網站
date: 2025-12-28T10:37:36+08:00
draft: false
aliases:
  - 手把手自架個人網站｜Hugo × GitHub Pages – Part 2：首次上線，透過 GitHub Actions 自動部署 Hugo 網站
description: 一步步教你使用 Hugo 搭配 GitHub Pages 與 GitHub Actions，自動化部署靜態網站，並說明與傳統手動方式的差異、主站與專案網站規則、快取設定，以及日後更換 Hosting 的實務重點。
slug: self-hosted-website-with-hugo-and-github-pages-part-2
categories: hugo-self-hosted-website
tags:
  - hugo
  - github_pages
  - personal_website
  - github_actions
---
> 上一篇用 Hugo 在本地端跑起了第一個網站。
>
> 👉 [Part 1：用 Hugo 在本地跑起第一個網站](/posts/self-hosted-website-with-hugo-and-github-pages-part-1/)
>
> 這次把 `Weisblog/` 放上 GitHub Pages，再用 GitHub Actions 自動編譯與部署。之後寫完文章，預覽沒問題就能推送上線。

## 一、GitHub Pages 是什麼？

GitHub Pages 是 GitHub 提供的靜態網站託管服務。如果把 Hugo 產生的 `public/` 比喻成房子，GitHub Pages 就是放房子的土地。免費的土地，還不香嗎？XD

![](assets/index/file-20251228215839068-e10dd386-57bc-47c8-85aa-eb447036dee7.png)

## 二、建立 Weisblog Repo，先分清楚網址

到 [GitHub](https://github.com/) 新增一個公開的 Repo，名稱填 `Weisblog`。這是本篇的教學範例名稱；下面所有 `<username>` 都要換成你自己的 GitHub 帳號。

![GitHub 新增 Repo 入口](assets/index/file-20251226161539378-f29337d4-dd0a-4e5a-b139-13cba3740b86.png)

*GitHub 首頁的 New 入口。*

GitHub Pages 的預設網址取決於 Repo 名稱：

| 類型 | Repo 名稱 | 預設網址 |
| --- | --- | --- |
| 帳號主站（user site） | `<username>.github.io` | `https://<username>.github.io/` |
| 專案網站（project site） | `Weisblog` | `https://<username>.github.io/Weisblog/` |

一個帳號只能有一個 user site；project site 則是每個 Repo 各自設定。這次用 **project site**，不必為了架部落格把 Repo 命名成 `<username>.github.io`。詳細命名規則可看 [GitHub Pages 說明](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)。

自訂網域設定好之後，同一個 project site 可以從 `https://blog.yungwei.com/` 提供內容，網址不再需要 `/Weisblog/`。這是下一篇要做的事，先不要把兩種路徑混在一起。

建立 Repo 時先不要勾選 README、license 或 `.gitignore`，這樣之後推送本地專案時，比較不會碰到兩邊各有一份初始紀錄的問題。

## 三、兩種部署方式，選一種就好

我一開始手動複製 `public/`，每次更新都要再跑一遍，後來改用 GitHub Actions，省事很多。建議直接走[方法二](#方法二github-actions-自動部署)；方法一保留給想先理解發布流程的人。

### 方法一：手動發布編譯結果

這個方式讓本地的 `Weisblog/` 保留 Hugo 原始碼，GitHub 的 `Weisblog` Repo 只存編譯後的網站。先替輸出建立另一個本地資料夾，避免跟原始碼混在一起。

在 `Weisblog/` 旁邊的上層目錄執行；指令中的帳號要先換成自己的：

```shell
mkdir Weisblog-public
cd Weisblog-public
git init
git branch -M main
git remote add origin "https://github.com/<username>/Weisblog.git"
cd ../Weisblog
```

> [!tip]
> 先將「這次要公開」的文章改成 `draft = false`，YAML frontmatter 則是 `draft: false`。其他草稿繼續保留，不要為了測試一次全部打開。

在 `config/_default/hugo.toml` 將既有的 `baseURL` 設成自己的 project site 網址：

```toml
baseURL = "https://<username>.github.io/Weisblog/"
```

下面的 `rsync --delete` 會清掉輸出資料夾裡已不在 `public/` 的檔案，所以目的地只能用剛建立的 `Weisblog-public/`，不要指向原始碼或其他專案。

```shell
hugo --cleanDestinationDir
rsync -av --delete --exclude='.git/' public/ ../Weisblog-public/
touch ../Weisblog-public/.nojekyll
cd ../Weisblog-public
git add .
git commit -m "Publish Hugo site"
git push -u origin main
```

回到 Repo 的 `Settings` → `Pages`，Source 選 **Deploy from a branch**，分支選 `main`、資料夾選 `/ (root)`。`.nojekyll` 是讓 GitHub 直接提供這些靜態檔案，不再當作 Jekyll 專案處理。

等部署成功後，打開 Pages 頁面提供的網址，並檢查 `/Weisblog/posts/` 與圖片是否正常。更新時再從 Hugo 專案跑一次編譯、同步與推送即可。

如果採用這個方法，先停在這裡。要改用 Actions，需先把部署 Repo 內容換成 Hugo 原始專案，保留 `.git` 歷史並檢查差異；不要把下面的指令直接接在輸出資料夾裡執行。

### 方法二：GitHub Actions 自動部署

這個方法把整個 Hugo 原始專案 `Weisblog/` 推到同名 Repo，再交給 GitHub 編譯與發布；本地 `public/` 不用上傳。[Hugo 官方部署文件](https://gohugo.io/host-and-deploy/deploy-to-github-pages/) 也是這個流程。

以下從「只完成 Part 1、還沒用方法一推送」的狀態開始。在 `Weisblog/` 建立 `.gitignore`：

```gitignore
public/
resources/
.hugo_build.lock
.DS_Store
```

上一篇安裝主題時已執行過 `git init`。確認自己在 Hugo 專案目錄，再連到剛建立的空 Repo：

```shell
git remote -v
git remote add origin "https://github.com/<username>/Weisblog.git"
git branch -M main
git add .
git commit -m "Add Hugo project"
git push -u origin main
```

`git remote -v` 如果已經列出 `origin`，先核對是否是這個 Repo；正確就略過 `remote add`。確定要更換才用 `git remote set-url origin <自己的Repo網址>`，避免推錯地方。

回到 `Weisblog` Repo → `Settings` → `Pages` → `Build and deployment`，把 Source 選成 **GitHub Actions**。不需要先讓 Jekyll 跑一次，也不必把首次失敗當作正常步驟。

![GitHub Pages Source 選擇](assets/index/file-20251229223057206-47cbf2df-95a4-4a52-ac49-bc1277d5830e.png)

*舊畫面：Repo 名稱不同，這裡選 GitHub Actions。*

在本地 `Weisblog/` 建立流程檔：

```shell
mkdir -p .github/workflows
touch .github/workflows/hugo.yaml
```

將下面內容存進 `.github/workflows/hugo.yaml`。這份範例固定使用 Hugo 0.168.0，符合目前 Blowfish 的最低需求；不代表這個版本永遠最新。日後若本機與主題需要較新版本，請同步調整 `HUGO_VERSION` 與下載檔名，其他依賴也依主題需求核對。建置時區使用台北。

```yaml
name: Build and deploy
on:
  push:
    branches:
      - main
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
defaults:
  run:
    shell: bash
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      DART_SASS_VERSION: 1.97.1
      GO_VERSION: 1.25.5
      HUGO_VERSION: 0.168.0
      NODE_VERSION: 24.12.0
      TZ: Asia/Taipei
    steps:
      - name: Checkout
        uses: actions/checkout@v5
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: false
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Create directory for user-specific executable files
        run: |
          mkdir -p "${HOME}/.local"
      - name: Install Dart Sass
        run: |
          curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "${HOME}/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}"
      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "hugo_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"
      - name: Verify installations
        run: |
          echo "Dart Sass: $(sass --version)"
          echo "Go: $(go version)"
          echo "Hugo: $(hugo version)"
          echo "Node.js: $(node --version)"
      - name: Install Node.js dependencies
        run: |
          if [[ -f package-lock.json || -f npm-shrinkwrap.json ]]; then
            npm ci
          fi
      - name: Configure Git
        run: |
          git config core.quotepath false
      - name: Cache restore
        id: cache-restore
        uses: actions/cache/restore@v4
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys:
            hugo-
      - name: Build the site
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
      - name: Cache save
        id: cache-save
        uses: actions/cache/save@v4
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}
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
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

對，我知道很長。XD 先認得幾個重點就好：Checkout 連主題 submodule 一起抓下來、Build 產生 `public/`、Upload 上傳網站成品、Deploy 才正式發布。`--baseURL` 使用 Pages 回傳的網址，會跟著 project site 或 Custom domain 設定走。

同時到 `config/_default/hugo.toml` 更新既有的 `baseURL`，讓本地正式編譯也使用正確網址；若前面已經有這個鍵，就修改原值，不要重複新增。尚未設定自訂網域時填：

```toml
baseURL = "https://<username>.github.io/Weisblog/"

[caches]
  [caches.images]
    dir = ':cacheDir/images'
```

`caches.images` 用來保存 Hugo 的圖片處理結果，像 Resize、Fit、Fill 就不必每次從頭跑；流程裡的 Cache restore／save 則讓不同次 Actions 建置共用快取。

> [!tip]
> 準備公開的文章設成 `draft: false`（TOML 是 `draft = false`），其餘草稿維持原值。

> [!important]
> 先跑 `hugo server` 檢查公開內容，若要看草稿才加 `-D`；正式 Actions 不加 `-D`。

確認後推送：

```shell
git add .
git commit -m "Add Pages deployment workflow"
git push
```

到 Actions 檢查 build 與 deploy 是否都成功，再打開 Pages 提供的網址。綠色勾勾表示流程執行成功，頁面、選單與圖片還是要自己點過；流程不會替你判斷文章寫得對不對。

![GitHub Repo 的 Deployments 區域](assets/index/file-20251229224715460-c3df885a-5c7f-4d2f-9510-390edbd29c20.png)

*舊畫面：Repo 名稱不同，部署結果看右側 Deployments。*

之後更新只要在本地預覽、確認提交內容，再推送即可。如果網址下有 `/Weisblog/`，文章連結與資產也要保留這段前綴；模板內產生連結可使用 Hugo 的 `relURL`／`relref`，避免硬寫 `/posts/` 直接跳到帳號主站。

#### 錯誤排除

我曾遇過推送 workflow 時，GitHub 拒收並提示 classic token 缺少 `workflow` scope：

```text
refusing to allow a Personal Access Token to create or update workflow
.github/workflows/hugo.yaml without workflow scope
```

若你也用 HTTPS 加 classic token，到 GitHub 個人 `Settings` → `Developer settings` → `Personal access tokens` 檢查 `repo` 與 `workflow` scope；fine-grained token 則核對目標 Repo 的 Contents 與 Workflows 寫入權限。這和網頁上 Actions 工作執行失敗是兩件事，先看錯誤發生在推送還是 build。

macOS 若一直使用舊憑證，可以在「鑰匙圈存取」找到對應帳號的 `github.com` 項目更新，再重新推送。我之前也用過 `security delete-internet-password -s github.com` 清除舊認證；有多個 GitHub 帳號時，先在鑰匙圈辨認要更新的那一筆，避免刪錯。Token 只會完整顯示一次，存好，別貼進 Repo。

## 四、換成自己的網址

完成後你會有一個 `https://<username>.github.io/Weisblog/` 的網站。下一篇接著把它接到 `blog.yungwei.com`，網站名稱使用 **Was Wei weiß**；這裡的網址是設定範例，請換成自己持有的網域。

> 👉 [Part 3：Namecheap 註冊、Cloudflare DNS 與 GitHub Pages 自訂網域](/posts/self-hosted-website-with-hugo-and-github-pages-part-3/)

### 之後換 Hosting 會很麻煩嗎？

原本的 Markdown 與圖片可以保留，再調整部署流程、`baseURL` 和 DNS。用新平台的內建建置時，只停用這個 Pages workflow；不要直接刪掉整個 `.github/workflows/`，裡面可能還有其他工作。網址或路徑也改了的話，還要處理舊連結轉址與搜尋引擎收錄，下一篇與 Part 4 會接著說明。
