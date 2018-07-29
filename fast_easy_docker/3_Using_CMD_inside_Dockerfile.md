# Using CMD inside Dockerfile

Up to this point we've been running our Docker containers by specifying the command in the command line:

```
$ docker run -p 3000:8000 -it eae332462b24 node server.js
```

But we can simplify this a little using the `CMD` command within our `Dockerfile`:

```
CMD [ "executable", "parameter" ]
```

So in our case, we specify:

```
FROM node:8.11.3-alpine

WORKDIR /app

ADD . /app

CMD [ "node", "server.js" ]
```

Any change to the `Dockerfile` means we have to rebuild the container. So:

```
docker build . -t belcurv-node:latest
```

We no longer need the interactive terminal (the `-it` flag), but we can specify the `-d` flag which means when we start up this container, it will automatically run in the background.

Then we can run the container:

```
docker run -d -p 3000:8000 eae332462b24
```

`-d` returns control to the terminal.
