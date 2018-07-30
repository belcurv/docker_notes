# Getting Started With Docker

Notes from: https://www.youtube.com/watch?v=Se-IUjRQe2c

Installation instructions:

https://docs.docker.com/install/linux/docker-ce/ubuntu/

DEBs:

https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/

Confirm Docker engine is running:

```
$ docker --version

> Docker version 17.12.1-ce, build 7390fc6
```

Likely have to add your current user to the `docker` group:

```
sudo usermod -a -G docker $USER
```

### Building a Docker Image

A `Dockerfile` (case sensitive, no file extension) tells the Docker engine what to include in your Docker image.

99% of Docker images will want to include a **base image**. We do this using the `FROM` command.

```
FROM [base_image]
```

We can find Docker base images in the Docker hub repository:

https://hub.docker.com/explore/

Here is a list of curated Docker images that the Docker company supports.

Each image is really a list of variations - various version numbers and bundled OSes. For help selecting a Node.js image for Docker, see this blog post:

https://derickbailey.com/2017/03/09/selecting-a-node-js-image-for-docker/

We'll use Node 8.11.3 on Alpine linux:

```
FROM node:8.11.3-alpine
```

That's actually all we need to get started - now we can build an image. The `docker build` command needs the path to the `Dockerfile`. Assuming you're in the same folder as your `Dockerfile`:

```
docker build .
```

This will download the base image and build the Docker image.

Then to see a list of Docker images:

```
$ docker images

REPOSITORY     TAG               IMAGE ID         CREATED         SIZE
node           8.11.3-alpine     ca148a52ea10     3 weeks ago     68.1MB
```

We could have passed in additional arguments, including tags. Tags can be used to give custom names to our images. Not specifying tags, Docker uses the base image tags from the `Dockerfile`. But we could have done:

```
docker build . -t belcurv-node:latest
```

The new build happens VERY quickly because it's really just a re-tag:

```
 $ docker images
REPOSITORY       TAG               IMAGE ID        CREATED         SIZE
belcurv-node     latest            ca148a52ea10    3 weeks ago     68.1MB
node             8.11.3-alpine     ca148a52ea10    3 weeks ago     68.1MB
```

### Run a Docker Image

Specify an image by its ID

```
docker run ca148a52ea10
```

Our current demo container will run and shutdown immediately because we haven't given it a process. There are no commands yet. Since Docker containers revolve around commands/processes, ours won't do anything at all.

But we can pass additional arguments to `docker run` to use some baked in functions of our image. For example, we can run the image with a base shell:

```
docker run -it ca148a52ea10 /bin/sh
```

That will give us a shell prompt within our Docker container. The container will run as long as the `/bin/sh` process runs. So if we `exit`, the container will shut down.

While a container is running, we can

```
docker ps
```

And see a list of running Docker containers.

### Something more interesting

Let's make a simple Node app.

```js
'use strict';

const http = require('http');

http.createServer((req, res) => {
  res.write('Hello world');
  res.end();
}).listen(8000);
```

So now we have to get the Node app into our Docker image. Back to the `Dockerfile`. After the base image, we have to set our `WORKDIR`: the working directory for all subsequent commands in the `Dockerfile`.

```
FROM node:8.11.3-alpine

WORKDIR /app
```

If the `/app` directory doesn't exist, Docker will create it. Now we'll copy our files into it. The `ADD` command takes 2 parameters, a source and a destination directory. We want to copy from `.` to the new `/app` directory:

```
FROM node:8.11.3-alpine

WORKDIR /app

ADD . /app
```

That's it. Now we can rebuild:

```
docker build . -t belcurv-node:latest
```

Now if we `docker images` we'll see our new image and that the ID has changed.

```
$ docker images
REPOSITORY       TAG               IMAGE ID         CREATED           SIZE
belcurv-node     latest            eae332462b24     20 seconds ago    68.1MB
node             8.11.3-alpine     ca148a52ea10     3 weeks ago       68.1MB
```

Now let's run the new image, with the shell process again:

```
$ docker run -it eae332462b24 /bin/sh
```

That will dump us at the `/app` path in our container. If we `ls` we'll see the same two files that we told Docker to copy into the `/app` directory (`Dockerfile` and `server.js`).

We could launch `server.js` from the container's shell, but let's just run it directly.

BUT - since Docker containers run in a virtual network, we have to map a local port to our container. We do that with the `-p` flag. The syntax is `local_port:container_port`, so we'll map local port 3000 to our server's listening port 8000:

```
$ docker run -p 3000:8000 -it eae332462b24 node server.js
```

We can "detatch" our terminal from the currently running container by holding down `Ctrl` and pressing `p` `q`

We can verify with `docker ps` - we'll see our node server is running.

We can stop it with `docker stop [container ID or the NAME]`

```
docker stop ddc21115ade2
```

or

```
docker stop peaceful_poitras
```

That usually takes a second or two ...

