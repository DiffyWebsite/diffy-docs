---
description: A way to remove cookie popups, mask highly dynamic elements like ads.
---

# Mask & Exclude

### Dynamic elements

Whenever you have highly dynamic elements on your pages you will soon notice that they give you false positives regularly. For example, you've added the Disqus widget or you have a slider or a video that autoplays.

For this, we have developed capabilities of masking (filling a region with green color) or excluding (completely removing from DOM) elements. You can find these settings under Modify section of the project's settings.

### &#x20;Examples

Check the block in the sidebar with a graph.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### Use Cases

You will need to use CSS selectors (the same as jQuery uses) to let the system know what elements to mask or exclude.

Good practices to mask would be elements like sliders, auto-played videos, ads.

Good examples of removing could be different popups (like cookies policy), live chats, and any other elements that won't break the layout if you remove them.
