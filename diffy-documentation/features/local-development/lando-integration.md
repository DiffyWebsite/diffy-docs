# Lando integration

You can take screenshots from your local Lando-powered website and upload them to Diffy. Here are a few steps.

### Snippet to your `.lando.yml` file

The snippet to add is:

```
services:
  diffy:
    type: compose
    webroot: .
    build:
      - rm -rf /app/.diffy-worker
      - mkdir -p /app/.diffy-worker
      - cd /app/.diffy-worker && wget -qO- https://github.com/DiffyWebsite/diffy-worker/archive/refs/heads/chrome136.tar.gz | tar xz --strip-components=1
      - cd /app/.diffy-worker && npm install
    services:
      image: diffywebsite2/screenshot-worker:0.0.4
      command: app
tooling:
  screenshot:
    service: diffy
    cmd: cd .diffy-worker && node --env-file=.env diffy-screenshots --url=https://diffy-marketing.lndo.site --screenshot-name=lando
```

This will add a container `diffywebsite2/screenshot-worker:0.0.4` to your setup and download the worker code to `.diffy-worker` directory.

Please pay attention that you need to replace `diffy-marketing.lndo.site` it with whatever your local environment URL is. Also, if you lik,e you can name your screenshots somehow differently than just `lando`.

### Provide the API Key and Project ID to `.env` file

Inside of `.diffy-worker` directory, you need to create a `.env` file (copy it from `.env.example`) and provide `DIFFY_API_KEY` and `DIFFY_PROJECT_ID`. These are to be obtained from the Diffy application. You can generate the API Key on [the Keys page](https://app.diffy.website/#/keys).

### Run screenshots

To run the screenshots, you can call

```
lando screenshot
```

This will take screenshots and upload them to Diffy.

Here is a video walkthrough of setting up a project and taking screenshots from your Lando site



{% embed url="https://www.loom.com/share/780b4a107bcb48f7b5aa6d9fc9810184" %}
