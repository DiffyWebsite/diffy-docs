---
description: Avoid Diffy hits reflected in your analytics, device identification.
---

# HTTP headers

You can set up HTTP headers for Diffy workers. We have a few cases when it is really useful.

### Google Analytics

If your site doesn't get much traffic Diffy's scans will be noticeable. To avoid that you can set

```
User-Agent: Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)
```

### Native device detection

If your layouts use device detection (instead of screen width) you can mimic different devices with User-Agent headers as well. There is a [great collection of devices/headers in puppeteer project](https://github.com/puppeteer/puppeteer/blob/main/packages/puppeteer-core/src/common/Device.ts).
