---
title: 手把手自架個人網站｜Hugo × GitHub Pages – Part 4：Google Analytics 與 Search Console 設定，追蹤流量並讓 Google 正確收錄網站
date: 2026-01-10T14:53:32+08:00
draft: false
aliases:
  - 手把手自架個人網站｜Hugo × GitHub Pages – Part 4：Google Analytics 與 Search Console 設定，追蹤流量並讓 Google 正確收錄網站
description: 一步步教你在 Hugo 靜態網站中設定 Google Analytics 與 Google Search Console，追蹤網站流量與使用者行為，並協助 Google 更快且正確地收錄網站內容。
slug: self-hosted-website-with-hugo-and-github-pages-part-4
categories: hugo-self-hosted-website
tags:
  - hugo
  - github_pages
  - personal_website
  - google_analytics
  - google_search_console
---
> 前一篇把 GitHub Pages 接上自己的網域。
>
> 👉 [Part 3：Namecheap 註冊、Cloudflare DNS 與 GitHub Pages 自訂網域](/posts/self-hosted-website-with-hugo-and-github-pages-part-3/)
>
> 網站能打開之後，我還想知道有沒有人來看、大家從哪裡找到文章。這篇接著設定 Google Analytics 4（GA4）與 Google Search Console，範例網址統一用 `https://blog.yungwei.com/`，網站名稱是 Was Wei weiß。

## 一、GA4：觀察流量與使用者行為

![](assets/index/file-20260110181515777-80042b3f-d98a-4067-8639-597c19106c40.png)

GA4 可以看流量來源、瀏覽頁面與互動事件。先到 [Google Analytics](https://analytics.google.com/analytics/) 登入，找出自己要管理的網站。

### 已經有 GA4，就先找既有 property 與 stream

網站換名字或網域時，先沿用這個網站既有的 property（資源）與 web data stream（網站資料串流），不要只因為名稱不同就另開一份，讓後續資料分散。

1. 在資源選單選對原本追蹤這個網站的 GA4 property。
2. 到「管理」→「資料收集與修改」→「資料串流」，開啟既有的網站串流。
3. 編輯串流詳細資料，網站網址使用 `https://blog.yungwei.com/`，串流名稱可改成 `Was Wei weiß`。
4. 記下同一份串流的評估 ID（Measurement ID），格式是 `G-XXXXXXXXXX`；Hugo 也繼續使用這組 ID。

![GA4 串流詳細資料中的評估 ID](assets/index/file-20260105222421003-dda13e25-772a-4e2b-9a10-90c15a822f69.png)

*舊畫面：網域與串流名稱不同，評估 ID 在右上角。*

資源與串流名稱是辨識用的文字，真正接收事件的是安裝在網站上的 ID。改後台網址不會替網站自動裝好追蹤碼。串流修改位置可對照 [Google 官方說明](https://support.google.com/analytics/answer/9304776?hl=zh-Hant)。

### 第一次使用，才建立新的資源與串流

如果原本完全沒有 GA4，照[首次設定流程](https://support.google.com/analytics/answer/9304153?hl=zh-Hant)建立帳戶、property，再新增 Web 串流。帳戶名稱可以自行取，時區與幣別依自己的報表需求填寫：

![GA4 新增網站串流的平台選擇](assets/index/file-20260105222339657-47516605-a135-4d89-aa05-18912e765572.png)

*第一次新增串流時選「網站」（Web）。*

| 項目 | 本篇範例／填寫方式 |
| --- | --- |
| Property 名稱 | `Was Wei weiß`，或自己認得的網站名稱 |
| 平台 | Web／網站 |
| 網站網址 | `https://blog.yungwei.com/`；介面若將協定拆開，選 HTTPS，網址欄只填 `blog.yungwei.com` |
| 串流名稱 | `Was Wei weiß` |
| 評估 ID | 使用系統提供的 `G-...`，不要照抄文章的佔位值 |

![GA4 網站網址與串流名稱欄位](assets/index/file-20260105222346201-a6b2d8c4-1c79-47dc-9a41-f397b28eca9a.png)

*舊畫面：網址與串流名稱不同，請填自己的正式網址。*

我當初直接拿網站網域當 property 名稱，也可以；名稱不需要跟網址綁死。原本就有帳戶的話，也不用為了每個網站重建帳戶。

## 二、把評估 ID 接到 Hugo

以 Blowfish 為例，不需要自己改主題的 `<head>` 或再貼一份完整 GA script。在 `Weisblog/config/_default/hugo.toml` 設定：

```toml
[services]
  [services.googleAnalytics]
    id = "G-XXXXXXXXXX"
```

將佔位值換成自己的評估 ID。若檔案已經有 `[services]` 或 `[services.googleAnalytics]`，就編輯原本的區段，不要重複宣告。設定名稱可對照 [Hugo services 文件](https://gohugo.io/configuration/services/)與 [Blowfish 設定說明](https://blowfish.page/docs/configuration/)。

### 主題怎麼把設定變成追蹤碼？

我當時檢查 Blowfish 的 `layouts/partials/head.html`，看到正式環境才載入分析功能：

```go-html-template
{{/* Analytics */}}
{{ if hugo.IsProduction }}
  {{ partial "analytics/main.html" . }}
{{ end }}
```

接著 `analytics/main.html` 會檢查是否有填 GA ID：

```go-html-template
{{ if site.Config.Services.GoogleAnalytics.ID }}
  {{ partial "analytics/ga.html" . }}
{{ end }}
```

`analytics/ga.html` 再將設定帶入 Google tag：

```html
<script async src="https://www.googletagmanager.com/gtag/js?id={{ site.Config.Services.GoogleAnalytics.ID }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', '{{ site.Config.Services.GoogleAnalytics.ID }}');
</script>
```

這些是主題原理的閱讀範例，**不用再貼進自己的文章或 layout**。主題版本更新時，檔案位置與內容可能調整，應以自己安裝版本為準。

一般的 `hugo server` 使用 development 環境；上面的正式環境判斷可避免本地預覽載入 GA。若自己指定 production 或另加了追蹤碼，本地測試仍可能送出事件。正式建置使用 production，發布前也要確認沒有把評估 ID 或追蹤功能關掉。

若已設定 Part 2 的 Actions，先預覽並核對這次提交的內容，再推送讓網站重新建置：

```shell
git add .
git commit -m "Configure Google Analytics"
git push
```

### 驗證事件是否送到正確串流

發布後，打開自己的正式網站，再回到同一個 GA4 property 的即時報表，觀察是否收到與測試時間相符的事件。必要時用 [Google Tag Assistant](https://tagassistant.google.com/) 核對 ID 與頁面網址，並留意是否重複安裝。

看到「已啟用資料收集」或活躍使用者大於 0，不能單獨證明新網域設定正確；那也可能是舊網址或其他頁面送來的資料。應把正確 ID、正式頁面和實際事件一起核對。阻擋追蹤的擴充套件、同意設定或報表延遲，也可能讓測試暫時看不到資料。

## 三、Search Console：檢查搜尋收錄

![](assets/index/file-20260110181658051-0179bb96-eeb5-4ca0-beb6-ddc12b16d2a8.png)

[Google Search Console](https://search.google.com/search-console/about) 用來查看搜尋曝光、點擊、索引狀態與網站問題。GA4 看使用者到站後的行為，Search Console 則能幫忙檢查 Google 搜尋這一端的狀況。

> [!info]- SEO 是什麼？
> SEO 是搜尋引擎最佳化，讓內容與網站結構更容易被搜尋引擎理解；完成驗證並不保證排名或立即收錄。

### 先檢查既有資源涵蓋哪裡

Search Console 也先找既有資源，但它的邊界跟 GA4 不同。GA4 可以延用原串流追蹤搬站後的網站，Search Console 的資源則依網域或網址前綴決定範圍：

| 既有資源 | 是否涵蓋 `https://blog.yungwei.com/` |
| --- | --- |
| Domain 資源 `yungwei.com` | 包含它的子網域與不同協定，可繼續使用 |
| Domain 資源 `blog.yungwei.com` | 包含這個子網域，可繼續使用 |
| URL-prefix `https://blog.yungwei.com/` | 包含這個 HTTPS 前綴，可繼續使用 |
| 其他網域，或 URL-prefix `https://yungwei.com/` | 不涵蓋這個部落格，需要適合新網址的資源 |

所以不用每次都重建資源，但也不能把「舊網域的資源」改個顯示名稱，就當成新網域已驗證。保留舊資源供查歷史資料，切換到涵蓋新網址的資源再操作。資源涵蓋規則見 [Google 官方說明](https://support.google.com/webmasters/answer/34592?hl=zh-Hant)。

### 沒有合適資源時，再新增與驗證

有 DNS 管理權限、想涵蓋整個網域時，新增 Domain 資源 `yungwei.com`，不含協定與路徑。若只想看部落格，也可以用 URL-prefix `https://blog.yungwei.com/`；仍使用 GitHub 預設 project site 的讀者，填 `https://<username>.github.io/Weisblog/`。

![Search Console 的兩種資源選項](assets/index/file-20260110162042342-0793a88a-eaa0-472c-97bf-294a6c15b665.png)

*舊畫面：網域不同，兩種資源的選擇位置相同。*

我先前使用自訂網域時採用 DNS 驗證。以 `yungwei.com` 的 Domain 資源為例，複製 Google 提供的驗證字串，到 **Cloudflare 的 `yungwei.com` zone** 新增：

![Search Console 提供的 TXT 驗證值](assets/index/file-20260110163755417-89196275-f37a-45e0-88f9-683606122229.png)

*舊畫面：網域及 DNS 業者不同，TXT 值請複製自己的。*

| 欄位 | 填寫內容 |
| --- | --- |
| Type | `TXT` |
| Name | `@`，代表這次驗證的根網域 |
| Content | Google 實際提供的 `google-site-verification=...` 全文 |
| TTL | `Auto` |

如果建立的是 `blog.yungwei.com` 的 Domain 資源，DNS 名稱要依該驗證畫面的指示設在 `blog`，不要一律填 `@`。TXT 是驗證記錄，不能拿前一篇路由用的 CNAME 取代，也不要覆蓋既有的其他 TXT。

回到 Search Console 點驗證；若暫時失敗，先核對 TXT 所在的名稱與內容、權威 DNS 是否已是 Cloudflare，再等解析更新。驗證完成後保留記錄，Google 可能再次檢查。

URL-prefix 可以使用 HTML tag、檔案或符合條件的 GA 等驗證方式；有 GA4 並不代表一定自動通過，仍須符合 Google 對帳號權限與追蹤碼的要求。依[擁有權驗證文件](https://support.google.com/webmasters/answer/9008080?hl=zh-Hant)選適合的方法即可。

## 四、提交 sitemap，確認實際文章狀態

> [!info]+ 什麼是 sitemap.xml？
> `sitemap.xml` 是提供給搜尋引擎的網址清單。
>
> Hugo 會產生 sitemap。一般可從網站根目錄的 `/sitemap.xml` 開始；多語網站可能是 sitemap index，再連到各語言的 sitemap。以本篇網址為例：
>
> ```text
> https://blog.yungwei.com/sitemap.xml
> ```

先用瀏覽器打開它，確認不是 404，也確認裡面的網址都使用正式網域。若本地正式編譯，對應輸出是 `public/sitemap.xml`；`baseURL` 設錯時，清單裡的網址也可能跟著錯。

到涵蓋新網址的 Search Console 資源，開啟「Sitemap」，填入完整 sitemap 網址並提交。看後續讀取狀態與錯誤，多語的子 sitemap 也要能正常讀取。

![Search Console 的 Sitemap 提交欄位](assets/index/file-20260110165207540-e1fe56ea-0ee8-475d-a086-901e9b9cbfe4.png)

*舊畫面：網域不同，提交欄位請填新站 sitemap 網址。*

Google 說明 [sitemap 有助於發現網址，但不保證抓取或建立索引](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview)。提交成功表示 Google 能處理這份清單，不能直接當成每篇文章都已收錄。

想檢查特定文章時，使用「網址審查」貼入正式文章網址，查看 Google 已知的索引狀態；需要時可測試線上網址、要求建立索引。剛上線的內容可能要等一段時間，沒有固定完成期限。

![Search Console 的網址審查與要求建立索引](assets/index/file-20260110171334688-507addf1-cd04-490c-b7ae-acc09b889847.png)

*舊畫面：網址不同，改貼要檢查的正式文章網址。*

你也可以在 Google 試搜：

```text
site:blog.yungwei.com
```

這能快速看看有哪些結果，但不是完整的收錄清單。是否收錄某篇文章，仍回 Search Console 看該網址的檢查結果。

如果正在換網域，除了新資源與 sitemap，也要保留舊資源、檢查舊文到新文的一對一轉址，以及 canonical、內部連結是否都使用新網址；符合條件時再依 [Google 搬站指南](https://developers.google.com/search/docs/crawling-indexing/site-move-with-url-changes)使用地址變更工具。不要把刪掉舊資源當作搬站步驟。

到這裡，從 Hugo 本地建站、GitHub Pages 部署、自訂網域，到流量與搜尋收錄的基本設定，就都有做法可循了。接下來我會繼續介紹 Blowfish 的介面排版與 Hugo 的進階用法。
