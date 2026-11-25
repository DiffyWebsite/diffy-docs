# Claude plugin

We have built a Claude plugin around our CLI tool. So now you can manipulate project settings and trigger screenshots and comparisons from Claude.

{% hint style="info" %}
You can even install a local worker (a Docker container) to take screenshots from your local dev environment using Claude.
{% endhint %}

Plugin link [https://github.com/DiffyWebsite/diffy-cli/tree/master/plugins](https://github.com/DiffyWebsite/diffy-cli/tree/master/plugins)

Basic setup could be as simple as:

```
/plugin marketplace add diffywebsite/diffy-skills
/plugin install diffy@diffy
```

And then you can use prompts like:

```
Create a project in Diffy with production URL https://diffy.website 
and pages /, /about-us, /features.

Take screenshots from my local site https://diffy.ddev.site, upload them to the project.

Take screenshots from the production environment and compare them with screenshots 
from my local site.

Show me the results and statistics and the URL to the report.
```

Also, you can do mass operations on project settings like

```
Edit all my projects settings add 1400px breakpoint to all of them.
```

Here is a demo of how to do all of this

{% embed url="https://www.loom.com/share/e1dd07ece4544900916983c59596b580" %}
