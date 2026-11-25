# Bypass Imperva Incapsula protection

If your project is protected by Imperva Incapsula, you will need to take a few extra steps to configure it.

1. Do not use any fixed delay in the project. We add delay dynamically once we see the Incapsula iframe, as these delays should be random.
2. Add the following headers:

```
Referer: https://your-homepage-url.com
Upgrade-Insecure-Requests: 1
Accept: text/ntml,agplication/xhtml+xml,agplication/xml:0=0.9,image/webg,/:g=0.8
Accept-Encoding: gzip, deflate, sach
Cache-Control: max-age=0 
```

Behind the scenes, we add a stealth plugin for puppeteer and also use the following JavaScript that emulates user behaviour and adds a random delay

```javascript
const handleIncapsula = async (page, maxRetries = 5) => {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    const iframeDetected = await page.$('iframe#main-iframe');
    if (iframeDetected) {
      const html = await page.content();
      const isIncapsula = html.includes('_Incapsula_Resource');

      if (isIncapsula) {
        logger.debug(`Incapsula iframe detected (attempt ${attempt + 1}/${maxRetries + 1})`);

        await page.mouse.move(300, 100);
        await page.mouse.click(300, 100);
        await page.keyboard.type('test');
        await page.keyboard.press('Tab');
        await page.evaluate(() => window.scrollBy(0, 100));
        await new Promise(resolve => setTimeout(resolve, 1000));

        const cleared = await page.waitForFunction(
            () => !document.querySelector('iframe#main-iframe'),
            { timeout: 10000 }
        ).catch(() => false);

        if (cleared) {
          logger.info('Incapsula iframe cleared. Proceeding...');
        }

        if (attempt < maxRetries) {
          logger.warn('iframe did not disappear. Retrying page reload...');
          await new Promise(resolve => setTimeout(resolve, 2000));
          await Promise.all([
            page.waitForNavigation({ waitUntil: 'networkidle2', timeout: 10000 }).catch(() => {}),
            page.reload()
          ]);
          await new Promise(resolve => setTimeout(resolve, 2000));
        } else {
          logger.error('Incapsula iframe still present after all retries.');
        }
      }
    }
  }
}
```
