# Misc javascript snippets

WP Engine replaces the link text automatically for staging environments. To avoid false positives that we can apply JavaScript:<br>

```
const allLinks = document.querySelectorAll('a');
allLinks.forEach(link => {
  let linkText = link.textContent;
  if (linkText.indexOf("abc.wpenginepowered.com") !== -1) {
    link.textContent = linkText.replace("abc.wpenginepowered.com", "abc.com");
  }
});
```
