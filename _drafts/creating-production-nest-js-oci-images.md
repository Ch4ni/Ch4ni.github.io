---
layout: post
title: Creating Production Nest.js OCI Images
tags:
  - node.js
  - next.js
  - docker
  - containers
  - optimization
---

In some of my previous roles I've been responsible for building and deploying container images for services running on Node.js. More specifically, I've been exposed to services that are written in Typescript using the [nest.js](https://nestjs.com/) framework.

It should come as no surprise that there are a fair number of back-end services out there written in Typescript, nor that there are folks who want to take advantage of container images (such as Docker images) to deploy their services to _some_ form of compute orchestration platform such as Kubernetes, or Amazon's ECS.

One of the things that is surprising, however, is just how large even a skeleton application's container image is when following along with the [nest.js documentation](docs.nestjs.com/deployment). I'm going to walk through a minimal example to reduce the size of a minimal nest.js app and explain some of what's happening along the way. My goal here is to provide an easy-to-follow example of some best-practices for creating production-ready images, as well as raising awareness for some of what's available with the tooling already on hand.

## The Set-Up

For this example I'm going to use [Node.js v24](https://nodejs.org/en/download/archive/v24.13.0), since it's the active LTS release at the time of writing.

Installing this version of node is beyond the scope of this article. I recommend creating a `.nvmrc` or `.node-version` file within the project directory containing only the major version for node and then using either [nvm](https://github.com/nvm-sh/nvm) or [mise-en-place](https://mise.jdx.dev/) configured to support [idiomatic version files](https://mise.jdx.dev/configuration.html#idiomatic-version-files).


To start, we'll make a new directory, set our node version, and then use the nest cli to initialize our project:

```shell
mkdir production-images && cd production-images
cat <<EOF>.node-version
24
EOF
npx -y @nestjs/cli new --package-manager npm --language TypeScript --directory . production-images
```

There we go! Now we have a project that we can build and run with `npm run build && npm start`!

## The Recommended Container Image

The Nest.js docs have a section on [Dockerizing your Application](https://docs.nestjs.com/deployment#dockerizing-your-application), so let's start with that.

As of the publication date, the sample Dockerfile in the the nest.js docs looked like this:

```Dockerfile

# Use the official Node.js image as the base image
FROM node:20

# Set the working directory inside the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install the application dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Build the NestJS application
RUN npm run build

# Expose the application port
EXPOSE 3000

# Command to run the application
CMD ["node", "dist/main"]

```


### Updating the Node Version

I'm going to start off with one very small change - the node version. I'm not going to just replace `20` with `24` though: I'm going to replace `20` with a [build arg](https://docs.docker.com/reference/dockerfile/#arg) that we can override via command-line.

So, replace these lines:
```Dockerfile

# Use the official Node.js image as the base image
FROM node:20
```

with these lines:

```Dockerfile
ARG NODE_VERSION=24

# Use the official Node.js image as the base image
FROM node:${NODE_VERSION}
```

This small update can allow us to avoid needing to change code to test against a newer version of node.js ... we can just run `docker build` with `--build-arg NODE_VERSION=<x>`. This is also available when using `podman build` ([link](https://docs.podman.io/en/v5.5.2/markdown/podman-build.1.html#build-arg-arg-value)) or `buildah build` ([link](https://github.com/containers/buildah/blob/main/docs/buildah-build.1.md?plain=1#L67:L75)) for CI/CD pipelines using any other container runtime than docker engine.

We should also include the following `.dockerignore` file to avoid copying unwanted files into the container image from our project repo:
```
node_modules
dist
*.log
*.md
.git
```


### Build the First Image

With this Dockerfile in place, we should now be able to build a container image:

```shell
docker build -t production-images:node24 .
```


Taking a look at the size we see that this image is ... almost 2GB! With no functionality!

```
$ docker images

IMAGE                      ID             DISK USAGE   CONTENT SIZE   EXTRA
production-images:node24   a4d496c1e1ee       1.97GB          468MB        
```

We can get a little more insight into what causes the image size by either diving in with a tool like [dive](https://github.com/wagoodman/dive), or by using the built-in `docker history`:

```
$ docker history production-images:node24
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
a4d496c1e1ee   2 minutes ago    CMD ["node" "dist/main"]                        0B        buildkit.dockerfile.v0
<missing>      2 minutes ago    EXPOSE [3000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      2 minutes ago    RUN /bin/sh -c npm run build # buildkit         279kB     buildkit.dockerfile.v0
<missing>      2 minutes ago    COPY . . # buildkit                             446kB     buildkit.dockerfile.v0
<missing>      18 minutes ago   RUN /bin/sh -c npm install # buildkit           283MB     buildkit.dockerfile.v0
<missing>      18 minutes ago   COPY package*.json ./ # buildkit                377kB     buildkit.dockerfile.v0
<missing>      3 hours ago      WORKDIR /usr/src/app                            16.4kB    buildkit.dockerfile.v0
<missing>      2 days ago       CMD ["node"]                                    0B        buildkit.dockerfile.v0
<missing>      2 days ago       ENTRYPOINT ["docker-entrypoint.sh"]             0B        buildkit.dockerfile.v0
<missing>      2 days ago       COPY docker-entrypoint.sh /usr/local/bin/ # …   20.5kB    buildkit.dockerfile.v0
<missing>      2 days ago       RUN /bin/sh -c set -ex   && export GNUPGHOME…   5.41MB    buildkit.dockerfile.v0
<missing>      2 days ago       ENV YARN_VERSION=1.22.22                        0B        buildkit.dockerfile.v0
<missing>      2 days ago       RUN /bin/sh -c ARCH= && dpkgArch="$(dpkg --p…   212MB     buildkit.dockerfile.v0
<missing>      2 days ago       ENV NODE_VERSION=24.13.0                        0B        buildkit.dockerfile.v0
<missing>      2 days ago       RUN /bin/sh -c groupadd --gid 1000 node   &&…   69.6kB    buildkit.dockerfile.v0
<missing>      3 days ago       RUN /bin/sh -c set -ex;  apt-get update;  ap…   619MB     buildkit.dockerfile.v0
<missing>      3 days ago       RUN /bin/sh -c set -eux;  apt-get update;  a…   194MB     buildkit.dockerfile.v0
<missing>      4 days ago       RUN /bin/sh -c set -eux;  apt-get update;  a…   52.2MB    buildkit.dockerfile.v0
<missing>      5 days ago       # debian.sh --arch 'amd64' out/ 'bookworm' '…   133MB     debuerreotype 0.17
```

There's a lot of really big layers here, which makes sense because this image uses `node:24` as a base - which has _a lot_ of things that we won't ever need for production images such as: development packages and headers, several build-chains, development tools, localization files, and much much more.

### First-Pass size reduction

Per the docker-node docs, there are three [image variants](https://github.com/nodejs/docker-node/tree/main?tab=readme-ov-file#image-variants) from which to choose when building a production-ready image:
- `node:<version>` - full-fat, debian-based default images that provide a node runtime.
- `node:<version>-alpine` - _the slimmest_ node runtime base. Comes with caveats.
- `node:<version>-slim` - debian-based images that contain a much smaller base system and provide a node runtime.

For the purposes of this exercise I'm going to stick with the `node:24-slim` versions. The reason for this is that it's smaller than the full-fat images, but is still built on top of glibc. When we switch over to Alpine there's a _good chance_ that everything will be working, but there _may_ be times where one needs to compile a library from source since Alpine is based on musl libc.

So, let's build with `node:24-slim`:
```shell
docker build -t production-images:node24-slim --build-arg NODE_VERSION=24-slim .
```

Once that's done building:
```shell
$ docker images 'production-images:*'

IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
production-images:node24        a4d496c1e1ee       1.97GB          468MB        
production-images:node24-slim   f277a962b151        667MB          139MB        

```

That's a pretty good improvement - the resulting image is almost 30% of the original size ... but we can do better.

## Improving On the Nest.js Recommendations

### Step One: _Deterministic_ Dependencies

CI/CD is an important part of software development and deployment. One of the principles that gave rise to CI/CD in the first place is: automation and deterministic behavior. If I run the same thing more than once with the same parameters in the same environment, I should get the same outputs _every time_.

Unfortunately, the recommended Dockerfile is not deterministic. The same build can be run on the same machine, with zero code changes, and generate a different artifact every time.

Fortunately, making the build more deterministic is relatively simple - just replace `npm install` with `npm ci`. `npm ci` is designed _specifically_ to be used in automated environments such is CI/CD and makes use of either `package-lock.json` or `npm-shrinkwrap.json` ([docs](https://docs.npmjs.com/cli/v11/commands/npm-ci)). This keeps dependencies locked to a specific version until the `package-lock.json` or `npm-shrinkwrap.json` is modified to a new version.
