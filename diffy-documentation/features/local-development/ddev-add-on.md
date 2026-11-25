# DDEV add-on

Our DDEV integration allows you to take screenshots from your local environment.



For that, you need to install DDEV add-on

```
ddev get diffywebsite/ddev-diffy
ddev restart
```

During the installaiton you will see the output of `npm install`. That is expected.

After successful installation you need to create `.env` file in `.ddev/diffy-worker` directory and populate it with DIFFY\_API\_KEY and DIFFY\_PROJECT\_ID.

And now you can run:

```
ddev screenshot
```

It will take screenshots from your environment and upload them to Diffy so you can compare them with other screenshots taken from any of your other environments (production, staging, development, or baseline).

See the video walkthrough.

{% embed url="https://www.loom.com/share/a3b750e32581458f9d2271969bba1bb8" %}



### Multiple domains

If you use multiple domains for your website i.e. you have `additional_hostnames` in your `config.yaml` file:

```
additional_hostnames:
  - sitea
  - siteb
```

You need to make sure `diffy` container knows how to talk to your site by using those domains. For that add `docker-compose.router.yaml` file to your `.ddev` directory with content:

```
services:
  diffy:
    external_links:
      - ddev-router:sitea.ddev.site
      - ddev-router:siteb.ddev.site
```

Then you can adjust your diffy command `.ddev/commands/diffy/screenshot` to use your custom domain name instead of simple `https://web`. So it will be:

```
node diffy-screenshots --url=https://sitea.ddev.site --screenshot-name=$DDEV_HOSTNAME
```
