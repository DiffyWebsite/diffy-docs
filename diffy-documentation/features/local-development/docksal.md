---
description: Create screenshots from your Docksal local site and upload them to Diffy.
---

# Docksal

{% embed url="https://www.loom.com/share/a4ca6228c37a41e0816f8f51754f327f" %}
video walkthrough of integration with Diffy
{% endembed %}

### Snippet to `.docksal/docksal.yml` file

You need to add a new service

<pre class="language-yaml"><code class="lang-yaml">services:
<strong>  # Diffy
</strong>  diffy:
    hostname: diffy
    image: diffywebsite2/screenshot-worker:0.0.4
    volumes:
      - ${PROJECT_ROOT}:/app:rw
    command: tail -f /dev/null
</code></pre>

This creates a container and mounts the whole project's directory under `/app` directory in the container.

Remember to restart the environment to pull the container.

### Install worker code

Now we can ssh to the container and install the worker's code

```bash
fin exec --in=diffy bash
su ubuntu
cd /app/ && mkdir .diffy-worker && cd .diffy-worker
wget -qO- https://github.com/DiffyWebsite/diffy-worker/archive/refs/heads/chrome136.tar.gz | tar xz --strip-components=1
npm install
```

### Provide the API Key and Project ID to `.env` file

Inside of `.diffy-worker` directory, you need to create a `.env` file (copy it from `.env.example`) and provide `DIFFY_API_KEY` and `DIFFY_PROJECT_ID`. These are to be obtained from the Diffy application. You can generate the API Key on [the Keys page](https://app.diffy.website/#/keys).

### Run screenshots

Run the screenshots command inside your container as `ubuntu` user:

```bash
cd /app/.diffy-worker
node --env-file=.env diffy-screenshots --url=https://docksal-demo.docksal.site --screenshot-name=docksal
```

Remember to adjust the `--url` for your local site's name and you can fine tune the name of the screenshots as well.
