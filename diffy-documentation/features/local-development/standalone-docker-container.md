# Standalone docker container

If your project has its docker-compose setup, you can add Diffy's container to the mix to take screenshots from your local environment. The following few steps will allow you to do this.

### Clone worker code

Clone the code from [https://github.com/DiffyWebsite/diffy-worker](https://github.com/DiffyWebsite/diffy-worker) to some directory in your project where it makes sense.

### Set up .env file

In the worker directory, copy `.env.example` to `.env` and edit `DIFFY_API_KEY` and `DIFFY_PROJECT_ID` variables.

The key can be generated under [My Account > Keys](https://app.gitbook.com/s/WkSLD59cnd4ChYfmqo3r/automation), and you need to grab project ID from the URL of your project.

### Add container

Add to your docker-compose.yml

```
    diffy:
        image: 'diffywebsite2/screenshot-worker:0.0.4'
        container_name: diffy
        command: tail -f /dev/null
        links:
            # This is link to your web server container. Can be different name.
            - webserver
        volumes:
            # Directory where you cloned worker code https://github.com/DiffyWebsite/diffy-worker
            - './diffy-worker:/diffy-worker'
```

Now remember to restart your container setup so the Diffy container is pulled in.

### Run \`npm install\`

SSH to Diffy container (`docker exec -it diffy bash`) and then in the worker code run `npm install` to pull all dependencies.

### Take screenshots from the container

Now you can  you can run screenshots:

```
node diffy-screenshots.js --url="https://webserver"
```

URL is the name of the container you linked in your docker-compose file

An important detail is that the screenshots command should be run as a non-root user! Otherwise, Chrome won't start properly. There is `ubuntu` user that you can use:

```
su ubuntu
```
