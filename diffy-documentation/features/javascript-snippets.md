---
description: Way to test you dropdown menus, popups and other interactive elements
---

# JavaScript snippets

Imagine you would like to take a screenshot of your page with the main menu expanded. You can do that by providing a little javascript snippet that will do exactly that right before we take the screenshot.

For that, we recommend using anchor tags so JavaScript knows when to run. So you have pages in the project:

```
https://example.com
https://example.com#top-menu
```

And then under the javascript section, you can target the anchor tag

```
if (window.location.hash == '#top-menu') {
  jQuery('header').find('.header__menu-toggle').click();
}
```

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>javascript snippets in action</p></figcaption></figure>
