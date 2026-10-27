---
title: "Hugo × GitHub Pages – Part 3: Namecheap, Cloudflare DNS, and a Custom GitHub Pages Domain"
date: 2025-12-30T21:39:31+08:00
draft: false
aliases:
  - "Hugo × GitHub Pages – Part 3: Custom Domains on GitHub Pages — Buying a Domain from Namecheap and DNS Configuration"
description: A practical guide to Namecheap registration, Cloudflare DNS, and a custom domain with HTTPS for the Weisblog project site on GitHub Pages.
slug: self-hosted-website-with-hugo-and-github-pages-part-3
categories: hugo-self-hosted-website
tags:
  - hugo
  - github_pages
  - personal_website
  - domain_name
---
> In Part 2, we put our website, the “house,” on GitHub Pages, the “land.”
>
> 👉 [Part 2: First deployment with GitHub Actions](/en/posts/self-hosted-website-with-hugo-and-github-pages-part-2/)
>
> Now we will give it an address: register a domain with Namecheap, let Cloudflare manage DNS, and connect the `Weisblog` project site to `blog.yungwei.com`.

## 1. What do domains, DNS, and hosting each do?

A domain name is like a street address: people can find your website without remembering an IP address. DNS resolves where that name points, while hosting provides the actual website files.

Here is the setup for this guide:

| Service | Responsibility | Example |
| --- | --- | --- |
| Namecheap | Domain registration, renewal, and nameserver settings | Own `yungwei.com` |
| Cloudflare | Authoritative DNS and its records | Create the `blog` CNAME |
| GitHub Pages | Website files and HTTPS | Publish the `Weisblog` repo |
| Hugo | Generate the site and content URLs | Set the production `baseURL` |

The registrar and DNS provider can be different services. Renewing a domain at Namecheap does not mean its DNS records must be edited there.

### Domain hierarchy

| Level | Example | Purpose |
| --- | --- | --- |
| Top-level domain (TLD) | `.com`, `.io`, `.org`, `.tw` | The rightmost label |
| Registered domain in this example | `yungwei.com` | The name you register and renew; `yungwei` is the second-level label |
| Subdomain | `blog.yungwei.com` | Separate services under the same domain |

You register `yungwei.com` and can add `blog` yourself, without buying another domain. We will configure only `blog.yungwei.com`. The apex `yungwei.com` and other subdomains can serve different purposes.

The project site from Part 2 defaults to `https://<username>.github.io/Weisblog/`. With a custom domain, visitors use `https://blog.yungwei.com/`, without `/Weisblog/`.

For me, the value of a custom domain is keeping control of the address I share. I can keep it if I change hosting providers. Preserving search performance also depends on page paths, redirects, and content; buying a domain alone does not guarantee rankings.

## 2. Register a domain with Namecheap

This part costs a little, but I think a nice address is worth it. Prices vary by extension, first-year offer, and renewal rate. Check the renewal price before paying, not just the introductory offer.

Search for your preferred domain on [Namecheap](https://www.namecheap.com/). You can also compare registrars such as [GoDaddy](https://www.godaddy.com/en) and [Gandi](https://www.gandi.net/en-US). `yungwei.com` is my configuration example; use a name you own or can register.

1. Check availability and add the name to your cart. Skip registration if you already own it.
2. Review the first-year and renewal prices, term, and privacy options you need.
3. You can enable Auto-renew and keep a valid payment method. Still keep track of the expiration date.
4. This GitHub Pages setup does not need an additional hosting package or SSL certificate; Pages provides HTTPS.
5. Keep the order confirmation and complete any contact-verification email required by the registrar.

“SSL certificate” remains a common label. Modern HTTPS uses TLS encryption to protect traffic between the browser and website.

## 3. Let Cloudflare manage DNS

If Cloudflare already manages your domain and its zone is Active, skip to the next section. **Do not reset its nameservers just to repeat the tutorial.**

For a new setup:

1. Sign in to [Cloudflare](https://dash.cloudflare.com/), add the whole `yungwei.com` zone, and choose a suitable plan. Add the apex domain here, not just `blog.yungwei.com`.
2. Review the scanned or imported records. Preserve existing sites, email MX/TXT records, and other subdomains; the automatic scan may miss records.
3. Note the two nameservers assigned to this zone. They can differ between domains, so do not copy someone else’s values.
4. In Namecheap, open `Domain List` → your domain’s `Manage` → `Nameservers`. Select **Custom DNS**, enter those Cloudflare nameservers, and save.
5. Wait for Cloudflare to show Active, then manage records in the zone’s `DNS` → `Records`.

If DNSSEC is already enabled, follow [Cloudflare’s full setup instructions](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/) to handle the old DS record before switching nameservers, then configure the new DNSSEC setup afterward. An old DS record that does not match the new DNS can prevent the entire domain from resolving. See [Namecheap’s nameserver instructions](https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-to-change-dns-for-a-domain/) for its interface.

Afterward, Namecheap handles registration and renewal, while DNS changes belong in Cloudflare. Namecheap’s `Advanced DNS` is no longer the active place to edit this domain’s records.

## 4. Add the custom domain in GitHub Pages first

Open the **`Weisblog` repo** on GitHub → `Settings` → `Pages`. Enter this under Custom domain:

```text
blog.yungwei.com
```

Enter only the hostname, without `https://` or `/Weisblog/`. Save it. A pending DNS check is expected at this point because we have not created the record yet. GitHub recommends adding the domain in Pages before pointing DNS to it. You can also [verify ownership at the account level](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages) first; that is separate from routing the site below.

### Add the CNAME in Cloudflare

In the `yungwei.com` zone’s `DNS` → `Records`, add:

| Field | Value |
| --- | --- |
| Type | `CNAME` |
| Name | `blog` |
| Target | `<username>.github.io`, using your own GitHub username |
| Proxy status | `DNS only` (gray cloud) |
| TTL | `Auto` |

**The CNAME Target is still the account’s `<username>.github.io`, regardless of the repo name.** Do not enter `Weisblog.github.io` or `https://<username>.github.io/Weisblog/`. A DNS record has no URL path. GitHub Pages uses its Custom domain setting to select which repository’s site serves that hostname. [GitHub’s custom domain documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) explicitly excludes the repository name from this Target.

This example uses DNS only, letting GitHub serve the website and HTTPS directly. If `blog` already has an A, AAAA, or CNAME record, identify its service before resolving the conflict. Leave the apex and other subdomains alone.

### How do A records and CNAME records differ?

An A record maps a name to an IPv4 address; a CNAME maps a name to another name. It is not a rule that A records are only for the apex or CNAME records only for `www`.

Our `blog` subdomain needs only the CNAME above. If you choose to put the **apex domain itself** on GitHub Pages, that is a different setup. One option uses GitHub’s four A records:

```text
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

For that setup, enter your apex domain in Pages and consider `www` DNS and redirects separately. Do not change `yungwei.com`’s A records just to configure `blog` in this guide.

## 5. Set Hugo’s URL and check HTTPS

Back in your local `Weisblog/`, update the existing value in `config/_default/hugo.toml`:

```toml
baseURL = "https://blog.yungwei.com/"
```

Keep the trailing `/`, without `/Weisblog/`. The Actions workflow in Part 2 builds with the `base_url` returned by Pages. Its Custom domain and this configuration should agree so canonical URLs, images, and sitemaps use the same address.

For manual deployment from Part 2, Pages adds a `CNAME` file to the publishing branch. Pull that commit first, and add `static/CNAME` to the Hugo source with one line, `blog.yungwei.com`, so future builds and syncs preserve it. With a custom Actions deployment, the Pages setting controls the domain and this file is not required.

Preview the site, review the changes, and push:

```shell
git add .
git commit -m "Set custom domain baseURL"
git push
```

With manual deployment, rebuild and sync as described in Method 1 of Part 2 before pushing. Do not push Hugo source directly into the publishing branch.

Back in Pages, wait for the DNS check and certificate provisioning, then enable **Enforce HTTPS**. GitHub Pages provides free HTTPS using [Let’s Encrypt](https://letsencrypt.org/) certificates. DNS and certificate issuance may take time, so the HTTPS option may not be available immediately.

If it remains unavailable, check the CNAME Target, gray-cloud status, the Pages domain, and [GitHub’s HTTPS troubleshooting instructions](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https). Do not repeatedly remove and re-add settings just because they have not taken effect yet.

When setup is complete, open your production address and check the homepage, posts, images, and language switch. Configuring `blog.yungwei.com` does not automatically make `yungwei.com` or `www.yungwei.com` the same website.

If you are moving an existing site to another domain, redirect old article URLs to their matching new URLs and keep slugs and paths where possible. DNS alone does not create HTTP redirects. Part 4 covers search indexing and traffic measurement for the new domain.

> 👉 [Part 4: Google Analytics and Search Console setup](/en/posts/self-hosted-website-with-hugo-and-github-pages-part-4/)
