---
title: 手把手自架個人網站｜Hugo × GitHub Pages – Part 3：Namecheap 註冊、Cloudflare DNS 與 GitHub Pages 自訂網域
date: 2025-12-30T21:39:08+08:00
draft: false
aliases:
  - 手把手自架個人網站｜Hugo × GitHub Pages – Part 3：GitHub Pages 自訂網域，從 Namecheap 購買自己的網路門牌與 DNS 設定教學
description: 從網域與 DNS 的基礎觀念開始，示範 Namecheap 註冊、Cloudflare DNS，以及 Weisblog project site 的 GitHub Pages 自訂網域與 HTTPS 設定。
slug: self-hosted-website-with-hugo-and-github-pages-part-3
categories: hugo-self-hosted-website
tags:
  - hugo
  - github_pages
  - personal_website
  - domain_name
---
> 上一篇把網站這棟「房子」放上 GitHub Pages 這塊「土地」。
>
> 👉 [Part 2：透過 GitHub Actions 自動部署 Hugo 網站](/posts/self-hosted-website-with-hugo-and-github-pages-part-2/)
>
> 這篇來替網站加上自己的門牌：在 Namecheap 註冊網域，交給 Cloudflare 管 DNS，再把 `Weisblog` project site 接到 `blog.yungwei.com`。

## 一、網域、DNS、Hosting 各管什麼？

網域（domain name）就像門牌，讓人不用記住 IP 位址也能找到網站。DNS 則負責查出這個名稱指向哪裡；網頁實際放在哪裡，由 Hosting 決定。

這次的分工是：

| 服務 | 負責的事 | 本篇範例 |
| --- | --- | --- |
| Namecheap | 網域註冊、續費、設定 nameserver | 持有 `yungwei.com` |
| Cloudflare | 權威 DNS，管理各個名稱的記錄 | 設定 `blog` 的 CNAME |
| GitHub Pages | 提供網站檔案與 HTTPS | 發布 `Weisblog` Repo |
| Hugo | 產生網站與內容網址 | `baseURL` 使用正式網址 |

註冊商與 DNS 服務可以分開。網域仍在 Namecheap 續費，不代表 DNS 記錄一定要在 Namecheap 改。

### 網域層級

| 層級 | 範例 | 用途 |
| --- | --- | --- |
| 頂級網域（TLD） | `.com`、`.io`、`.org`、`.tw` | 網域最右邊的一段 |
| 本例的註冊網域 | `yungwei.com` | 申請與續費的名稱；其中 `yungwei` 是二級標籤 |
| 子網域 | `blog.yungwei.com` | 在同一個網域下區分服務 |

你買的是 `yungwei.com`，`blog` 可以自己新增，不用另外買一個網域。這篇只設定 `blog.yungwei.com`；根網域 `yungwei.com` 和其他子網域可以有各自的用途。

上一章的 project site 預設是 `https://<username>.github.io/Weisblog/`。設定自訂網域後，讀者使用 `https://blog.yungwei.com/`，不需要再接 `/Weisblog/`。

我覺得自訂網域值得的地方，是分享出去的網址掌握在自己手上；以後搬 Hosting，仍能保留這個門牌。SEO 能不能延續，還要看頁面路徑、轉址與內容，光是買了網域並不保證排名。

## 二、在 Namecheap 註冊網域

這裡會需要小課金，為了漂亮的網址我覺得值得啦～不過價格會隨副檔名、首年優惠與續約費率變動，結帳前記得看「續約價」，不要只看第一年。

到 [Namecheap](https://www.namecheap.com/) 搜尋想要的網域。也可以比較 [GoDaddy](https://www.godaddy.com/en)、[Gandi](https://www.gandi.net/en-US) 等註冊商，依自己的需求選擇。以下的 `yungwei.com` 是我的設定範例，讀者要使用自己可註冊或已持有的名稱。

1. 確認名稱可註冊，再加入購物車；已經持有網域就不必重買。
2. 核對首年與續約費、註冊年限，以及需要的隱私保護選項。
3. 可開啟 Auto-renew，並確認付款方式有效；自動續費不代表可以完全忘記到期日。
4. 這個 GitHub Pages 流程不需要另外購買 Hosting 或 SSL 憑證，HTTPS 由 Pages 提供。
5. 付款後保留訂單，收到註冊聯絡資料驗證信時，依信件要求完成驗證。

一般仍會看到「SSL 憑證」這個名稱；現在 HTTPS 實際使用的是 TLS 加密，用來保護瀏覽器與網站之間的傳輸。

## 三、讓 Cloudflare 管 DNS

如果這個網域已經由 Cloudflare 管理，且狀態是 Active，可直接往下一節走，**不要為了照文章重設一次 nameserver**。

第一次設定時：

1. 登入 [Cloudflare](https://dash.cloudflare.com/)，新增整個 `yungwei.com` zone，依需求選方案；這裡新增的是根網域，不是只新增 `blog.yungwei.com`。
2. 核對掃描或匯入的 DNS 記錄。既有網站、郵件的 MX／TXT 與其他子網域都要保留；自動掃描不保證抓到全部。
3. 記下 Cloudflare 分配給這個 zone 的兩個 nameserver，每個網域分到的值可能不同，不要照抄別人的。
4. 到 Namecheap → `Domain List` → 該網域的 `Manage` → `Nameservers`，選 **Custom DNS**，填入那兩個 Cloudflare nameserver 後儲存。
5. 等 Cloudflare 顯示 Active，再到這個 zone 的 `DNS` → `Records` 管理記錄。

若原先已啟用 DNSSEC，更換 nameserver 前先依 [Cloudflare 完整設定說明](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/) 處理舊 DS 記錄，切換完成後再設定新的 DNSSEC；舊 DS 與新 DNS 不相符，可能讓整個網域無法解析。Namecheap 的介面位置可對照[官方 nameserver 說明](https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-to-change-dns-for-a-domain/)。

完成後 Namecheap 負責註冊與續費，DNS 的新增／修改都在 Cloudflare；Namecheap 的 `Advanced DNS` 不再是這個網域的生效位置。

## 四、先在 GitHub Pages 填自訂網域

先進 GitHub 的 **`Weisblog` Repo** → `Settings` → `Pages`，Custom domain 填：

```text
blog.yungwei.com
```

這裡只填主機名稱，不加 `https://`，也不加 `/Weisblog/`。儲存後 DNS 檢查暫時尚未通過沒關係，接下來才建立對應記錄。GitHub 建議先在 Pages 加入網域，再讓 DNS 指過去；也可依[官方說明](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)先做帳號層級的網域驗證，與下面的網站路由設定分開處理。

### 到 Cloudflare 新增 CNAME

在 `yungwei.com` zone 的 `DNS` → `Records` 新增：

| 欄位 | 填寫內容 |
| --- | --- |
| Type | `CNAME` |
| Name | `blog` |
| Target | `<username>.github.io`，換成自己的 GitHub 帳號 |
| Proxy status | `DNS only`（灰雲） |
| TTL | `Auto` |

**CNAME 的 Target 仍是帳號的 `<username>.github.io`，跟 Repo 叫什麼名字無關。** 不要填 `Weisblog.github.io`，也不要填 `https://<username>.github.io/Weisblog/`。DNS 記錄沒有網址路徑；由 GitHub Pages 的 Custom domain 設定，決定這個主機名稱要提供哪個 Repo 的網站。[GitHub 自訂網域文件](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)也特別說明 Target 不含 Repo 名稱。

這個範例使用 DNS only，讓 GitHub 直接處理網站與 HTTPS。如果同名 `blog` 已有舊 A、AAAA 或 CNAME，先確認原服務，再處理衝突；不要動到根網域或其他子網域的記錄。

### A Record 與 CNAME 怎麼分？

A Record 是名稱指向 IPv4，CNAME 是名稱指向另一個名稱；並不是「A 只能給根網域、CNAME 只能給 www」。

這次用 `blog` 子網域，只需要前面的 CNAME。若你選擇把**根網域本身**放在 GitHub Pages，才使用另一套設定，例如 GitHub 提供的四個 A Record：

```text
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

那種情況要在 Pages 填自己的根網域，並另考慮 `www` 的 DNS／轉址。不要為了本篇的 `blog` 設定，就把 `yungwei.com` 的 A Record 一起改掉。

## 五、設定 Hugo 網址，檢查 HTTPS

回到本地 `Weisblog/`，修改 `config/_default/hugo.toml` 的既有值：

```toml
baseURL = "https://blog.yungwei.com/"
```

正式網域後面保留 `/`，不用 `/Weisblog/`。若延用 Part 2 的 Actions，建置指令會使用 Pages 回傳的 `base_url`；GitHub 的 Custom domain 與這裡的設定應一致，避免 canonical、圖片或 sitemap 指向不同網址。

如果走的是 Part 2 手動部署，Pages 會在發布分支加入 `CNAME` 檔。先拉回那次提交，並在 Hugo 原始專案新增 `static/CNAME`，內容是一行 `blog.yungwei.com`，讓之後重新編譯、同步時仍能保留。若使用自訂 Actions workflow 發布，則以 Pages 設定為準，不需要靠這個檔案設定網域。

預覽與檢查提交內容後推送：

```shell
git add .
git commit -m "Set custom domain baseURL"
git push
```

手動部署的讀者則照 Part 2 方法一重新編譯與同步後推送，不是直接把 Hugo 原始碼放進發布分支。

回到 Pages，等 DNS 檢查與憑證準備完成，再勾 **Enforce HTTPS**。GitHub Pages 提供免費 HTTPS，背後使用 [Let’s Encrypt](https://letsencrypt.org/) 憑證；DNS 與憑證可能需要等待，HTTPS 選項也不一定立刻可用。

若一直無法啟用，先核對 CNAME Target、灰雲狀態、Pages 的網域與 [GitHub HTTPS 排錯說明](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)。不要只因為沒立刻成功，就反覆刪除再新增設定。

完成後打開自己的正式網址，實際檢查首頁、文章、圖片與語言切換。這篇設定的是 `blog.yungwei.com`，不代表 `yungwei.com` 或 `www.yungwei.com` 也自動變成同一個網站。

若是已有網站更換網域，還要把舊文章網址一對一轉到新網址，盡量保留 slug 與路徑；DNS 本身不會替你做 HTTP 轉址。新網域的搜尋收錄與流量資料，接著在 Part 4 處理。

> 👉 [Part 4：Google Analytics 與 Search Console 設定](/posts/self-hosted-website-with-hugo-and-github-pages-part-4/)
