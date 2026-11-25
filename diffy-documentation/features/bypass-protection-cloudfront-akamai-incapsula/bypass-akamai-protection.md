# Bypass Akamai protection

Taking screenshots from a protected website by Akamai CDN is pretty challenging. But here is a set of HTTP Headers you can set to bypass the protection:

```
User-Agent: Mozilla/5.0 (Linux; Android 11; SAMSUNG SM-G973U) AppleWebKit/537.36 (KHTML, like Gecko) SamsungBrowser/14.2 Chrome/87.0.4280.141 Mobile Safari/537.36
Accept-Language: en-au, en
Accept-Encoding: gzip, deflate, br
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Referer: https://google.com
```

You can set these under project settings > custom headers section.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Also, make sure to use a single worker and set the delay between runs. We had success with the 20s delay.

