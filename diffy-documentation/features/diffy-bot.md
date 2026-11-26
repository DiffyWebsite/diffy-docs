---
description: "What the Diffy bot is, what it does on your site, and how to identify or allow it."
---

# Diffy bot

Diffy is a visual regression testing service. To compare versions of a website, Diffy runs screenshot workers: headless browsers that open web pages and take full-page screenshots. This page explains how that bot works so site owners and network providers can identify its traffic.

### What the bot does

The Diffy bot opens pages that a Diffy customer has added to their project. For each page it:

1. Loads the page in a headless browser (Chrome or WebKit) at each configured screen width (breakpoint).
2. Waits for the page to render, including JavaScript, CSS, fonts and images.
3. Takes a full-page screenshot.

Diffy then compares these screenshots with earlier screenshots or with screenshots of another environment (for example, production versus staging) and shows the customer what changed visually.

The bot does not submit forms, buy anything, or change content. It only collects screenshots, and the screenshots are visible only to the customer who owns the project.

### Which sites it visits

The bot visits only URLs that a Diffy user has added to their own project, typically their own websites or their clients' websites. It does not crawl the web, index content, or collect data for search engines or AI training.

### When it runs

Screenshots are taken only when a project owner triggers them:

* **Manually** from the Diffy dashboard.
* **On a schedule** that the project owner sets, for example every few hours, daily, or on specific days.
* **From automation**, such as CI/CD pipelines (GitHub Actions, GitLab CI, CircleCI, Jenkins), the [Diffy CLI](https://github.com/DiffyWebsite/diffy-cli) or the Diffy API, usually after a deployment.

### Load on your site

Each run loads every configured page once per breakpoint. Project owners can limit the number of workers that hit their site at the same time, so a run does not overload it.

### How to identify the bot

<table><thead><tr><th width="200">Property</th><th>Value</th></tr></thead><tbody><tr><td>IPv4 address</td><td><code>138.201.56.149</code></td></tr><tr><td>IPv6 address</td><td><code>2a01:4f8:172:1189::2</code></td></tr><tr><td>User-Agent</td><td>The default User-Agent of the Chromium version the worker runs. Project owners can change it in project settings.</td></tr></tbody></table>

Requests come from this single IP address when the project has **Use single IP address** enabled in **Project Settings > Advanced > Optimization**. See [Bypass protection Cloudfront, Akamai, Incapsula](bypass-protection-cloudfront-akamai-incapsula/README.md).

{% hint style="info" %}
Project owners can set custom HTTP headers, including the User-Agent, for their own projects (see [HTTP headers](http-headers.md)). This is used, for example, to test device-specific layouts on sites they control.
{% endhint %}

### Allowing the bot

If your CDN or firewall blocks Diffy, allow the IP addresses above. For Cloudflare, see [Allow traffic from IP addresses in allowlist](https://developers.cloudflare.com/waf/custom-rules/use-cases/allow-traffic-from-ips-in-allowlist/).

### Contact

Diffy is operated by YG Technology Inc., Vancouver, Canada. If you have questions about Diffy bot traffic on your site, or want it to stop, email [info@diffy.website](mailto:info@diffy.website).
