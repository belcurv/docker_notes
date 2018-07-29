# Debugging broken Docker builds

Explores a useful tactic to debug broken Docker builds. TLDR: we run the _shell_ program inside a partially built Docker image, to inspect what is failing, and to figure out how to fix it.

Notes from: https://www.youtube.com/watch?v=RH_I0KXHBcA

Occasionally, we need to interactively debug shit inside a Docker image. OK, so we can launch an image with the interactive terminal using the `-it` flag:

```
docker run -it [IMAGE_ID] /bin/sh
```

And from there, we can use an editor to ... wait, we don't have any editors. Not even vim, apparently. This is alpine linux, afterall.

So, we can include a `RUN` command in the `Dockerfile` to install things during the build process.

For vim

```
FROM node:8.11.3-alpine

WORKDIR /app

ADD . /app

RUN apk add vim

CMD [ "node", "server.js" ]
```

What about nano? Let's try ... 

```
FROM node:8.11.3-alpine

WORKDIR /app

ADD . /app

RUN apk add nano

CMD [ "node", "server.js" ]
```

then

```
docker build . -t belcurv-node:latest
```

And we get a pile of errors.

```
$ docker build . -t belcurv-node:latest
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM node:8.11.3-alpine
 ---> ca148a52ea10
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> a835f6a39870
Step 3/5 : ADD . /app
 ---> 15b02f16859d
Step 4/5 : RUN apk add nano
 ---> Running in 596fc9a30ad1
WARNING: Ignoring APKINDEX.84815163.tar.gz: No such file or directory
WARNING: Ignoring APKINDEX.24d64ab1.tar.gz: No such file or directory
ERROR: unsatisfiable constraints:
  nano (missing):
    required by: world[nano]
The command '/bin/sh -c apk add nano' returned a non-zero code: 1
```

Whenever Docker builds an image, it actually builds all those 'step' intermediate images along the way. We have access to them.

So, to debug this error further we can run the image immediately before the error happens:

```
docker run -it 15b02f16859d /bin/sh
```

Now we can reissue the command that failed within the container's shell:

```
apk add nano
```

We can reproduce the error - in this case, `apk` is missing a flag:

```
apk --update add nano
```

And now we have `nano` (or whatever).  Of course, we really want this in our `Dockerfile`:

```
FROM node:8.11.3-alpine

WORKDIR /app

ADD . /app

RUN apk --update add nano

CMD [ "node", "server.js" ]
```

Now, we rebuild and Docker goes out and fetches `nano` during the build stage.

