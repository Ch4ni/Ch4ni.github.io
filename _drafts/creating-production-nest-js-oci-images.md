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

Let's use a new Dockerfile based on the previous step, this time with the `ci` dependency install:

```Dockerfile
ARG NODE_VERSION=24

# Use the official Node.js image as the base image
FROM node:${NODE_VERSION}

# Set the working directory inside the container
WORKDIR /usr/src/app

# copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install the (version locked) application dependencies
RUN npm ci

# Copy the rest of the application files
COPY . .

# Build the NestJS application
RUN npm run build

# Expose the application port
EXPOSE 3000

# Command to run the application
CMD ["node", "dist/main"]
```

This change doesn't save us any space, but it does make image builds reproducible, which allow us to make use of other techniques.

Before continuing, it's worth noting that the `NODE_VERSION` is at `24`, which generally means that it's not _pinned_ to a specific tag or release. In this case, this is by design - the version `24` will be used as the default for any local development builds which should be fine. To make builds both flexible _and_ reproducible, this should be specified as a parameter to the `docker build` via CI/CD ... there are a number of ways to fetch the appropriate version number, but that task lay outside the scope of this article.

### Step Two: Multi-Stage Builds

Splitting the build up into multiple stages allows for smaller final images, as well as the opportunity to make better use of layer caching. It's easy to go overboard with the number of stages, so I recommend starting with a small number and increasing the number of stages as it makes sense.

Here, we can start with a builder stage, and a final image:

```Dockerfile
ARG NODE_VERSION=24

# Use the official Node.js image as the base image
FROM node:${NODE_VERSION} AS builder

# Set the working directory inside the container
WORKDIR /usr/src/app

# copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install the (version locked) application dependencies
RUN npm ci

# Copy the rest of the application files
COPY . .

# Build the NestJS application
RUN npm run build

# This defines a new build stage for the final image
FROM node:${NODE_VERSION}

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app /usr/src/app

# Expose the application port
EXPOSE 3000

# Command to run the application
CMD ["node", "dist/main"]

```

Build the image:
```shell
docker build -t production-images:node24-slim-multistage . --build-arg NODE_VERSION=24-slim
```

And we can take a look at the new size:
```shell
$ docker images

IMAGE                                            ID             DISK USAGE   CONTENT SIZE   EXTRA
node:24                                          b2b2184ba9b7       1.64GB          424MB        
node:24-slim                                     bf22df20270b        326MB         81.7MB        
production-images:node24                         a4d496c1e1ee       1.97GB          468MB        
production-images:node24-slim                    f277a962b151        667MB          139MB        
production-images:node24-slim-npmci              8b2b9500774e        666MB          139MB        
production-images:node24-slim-npmci-multistage   457594ab46e8        594MB          111MB     

```

Overall we've managed to save 28MB from our image content, and overall saved about 70MB or 10% from total disk usage. That's not bad!

### Step Three: Runtime-Only Dependencies

Using the multi-stage build approach we can now build an image that specifically only installs _runtime_ dependencies. A basic image stage for that might look like this:

```Dockerfile
ARG NODE_VERSION=24
FROM node:${NODE_VERSION}
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --omit=dev
```


With the runtime build-stage defined, we will still need to copy over our built application to the new image/layer.

During the `npm run build` stage nest.js creates a `dist/` directory [and copies the resulting build into that directory](https://docs.nestjs.com/deployment) - we can use this to copy only the built application without any additional source code or other unnecessary files into the final image:

```Dockerfile
ARG NODE_VERSION=24

# Use the official Node.js image as the base image
FROM node:${NODE_VERSION} AS builder

# Set the working directory inside the container
WORKDIR /usr/src/app

# copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install the (version locked) application dependencies
RUN npm ci

# Copy the rest of the application files
COPY . .

# Build the NestJS application
RUN npm run build

# This defines a new build stage for the final image
FROM node:${NODE_VERSION}

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm ci --omit=dev && npm prune --omit=dev
COPY --from=builder /usr/src/app/dist /usr/src/app/dist

# Expose the application port
EXPOSE 3000

# Command to run the application
CMD ["node", "dist/main"]

```


Then build the image with:
```shell
docker build -t production-images:node24-slim-npmci-multistage-runtimedeps . --build-arg NODE_VERSION=24-slim
```

And inspect the new size:
```shell
$ docker images

IMAGE                                                        ID             DISK USAGE   CONTENT SIZE   EXTRA
node:24                                                      b2b2184ba9b7       1.64GB          424MB        
node:24-slim                                                 bf22df20270b        326MB         81.7MB        
production-images:node24                                     a4d496c1e1ee       1.97GB          468MB        
production-images:node24-slim                                f277a962b151        667MB          139MB        
production-images:node24-slim-npmci                          8b2b9500774e        666MB          139MB        
production-images:node24-slim-npmci-multistage               457594ab46e8        594MB          111MB        
production-images:node24-slim-npmci-multistage-runtimeOnly   641abbad4869        360MB         84.2MB        

```

Our new image is 234MB smaller on disk than the previous one - that's almost 40% smaller than the last technique!

### Step four: Squashing Layers

The next thing we can do to make our image smaller is to take some of the improvements from previous steps and combine them to remove layers from the final build. `docker build` used to have a `--squash` option, but that behavior was removed with the move to BuildKit. `podman build` still has support for the `--squash` [cli argument](https://docs.podman.io/en/stable/markdown/podman-build.1.html#squash).

Since Docker build no longer supports `--squash` we can make use of build stages to perform a partial squash by assembling our specific changes and copying those en-masse over the base image:

```Dockerfile
ARG NODE_VERSION=24

# Use the official Node.js image as the base image
FROM node:${NODE_VERSION} AS builder

# Set the working directory inside the container
WORKDIR /usr/src/app

# copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install the (version locked) application dependencies
RUN npm ci

# Copy the rest of the application files
COPY . .

# Build the NestJS application
RUN npm run build

# This defines a new build stage for the runtime stage
FROM node:${NODE_VERSION} AS runtime

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm ci --omit=dev && npm prune --omit=dev
COPY --from=builder /usr/src/app/dist /usr/src/app/dist

# This defines a new build stage for the final image
FROM node:${NODE_VERSION}
WORKDIR /usr/src/app

# copy the assembled runtime build directly on top of the node layer
COPY --from=runtime /usr/src/app /usr/src/app

# Expose the application port
EXPOSE 3000

# Command to run the application
CMD ["node", "dist/main"]
```

Then build the new image:
```shell
docker build -t production-images:node24-slim-npmci-multistage-runtimeonly-squashed --build-arg NODE_VERSION=24-slim
```

And look at the new sizes:
```shell
$ docker images

IMAGE                                                                 ID             DISK USAGE   CONTENT SIZE   EXTRA
node:24                                                               b2b2184ba9b7       1.64GB          424MB        
node:24-slim                                                          bf22df20270b        326MB         81.7MB        
production-images:node24                                              a4d496c1e1ee       1.97GB          468MB        
production-images:node24-slim                                         f277a962b151        667MB          139MB        
production-images:node24-slim-npmci                                   8b2b9500774e        666MB          139MB        
production-images:node24-slim-npmci-multistage                        457594ab46e8        594MB          111MB        
production-images:node24-slim-npmci-multistage-runtimeOnly            641abbad4869        360MB         84.2MB        
production-images:node24-slim-npmci-multistage-runtimeOnly-squashed   526308a8f6c3        350MB         81.4MB
```

This final step removes an additional 10MB from the image's disk usage, 2.8MB of which comes from savings in our application source. At this point our image is 350MB on disk (81.4MB of which is our application and dependencies) - down from 1.97GB on disk with 468MB of application and dependencies. This represents a total of about 17.7% of the total disk space for an image following the Nest.js recommendations.

## (Optional) Step Five: Test Building with Alpine

With all previous steps completed, we can test building the application with alpine by modifying a build-arg:

```shell
docker build -t production-images:node24-alpine-runtime . --build-arg NODE_VERSION=24-alpine
```

And we can see that this takes another 97MB off of the image bringing the total size down to just under 13% of the original image size!

```shell
$ docker images production-images:node24-alpine-runtime

IMAGE                                     ID             DISK USAGE   CONTENT SIZE   EXTRA
production-images:node24-alpine-runtime   e0097c5557c0        253MB         58.4MB       
```

Of course node on Alpine isn't _guaranteed_ to run every Nest.js app; there may even be build failures due to the underlying libc implementation - this is why Alpine comes as the _last_ step in optimizing for image size - we have good enough gains without any loss in compatibility.

## Conclusion

Using a combination of selecting [an appropriate base-image](https://github.com/nodejs/docker-node/blob/main/README.md#image-variants), [multi-stage docker builds](https://docs.docker.com/build/building/multi-stage/), [runtime-only dependencies](https://docs.npmjs.com/cli/v10/commands/npm-ci#omit), and layer squashing (a variation on multi-stage builds) we can easily cut approximately 80-87% from a Nest.js Container Image built with the Dockerfile recommended in the Nest.js documentation.

This article focused on doing this with `docker build` and `npm`, but there are other tools for generating Container images (such as `podman build` or `buildah`), or managing node.js builds and dependencies (such as `yarn` and `pnmp`). To make use of these techniques for different tools, please refer to your specific tool's documentation.

Optimizing container builds for disk size is only one facet of container optimization; there are no end of options, and no end of "micro-optimizations" - that's some of why I wrote this article: because I could see some low-effort ways to build smaller, more predictable production-ready artifacts than what's in the official Nest.js documentation.

I wrote this article to address some low-effort areas to optimize nest.js applications for production workloads. You can find some other best-practices and useful tools/techniques in some of the links below:

- docker-node (node.js official Docker Images) [best-practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md) (Github)
- docker [building best practices](https://docs.docker.com/build/building/best-practices/) (docs.docker.com)
- npm [documentation](https://docs.npmjs.com/cli/v11/commands/npm-ci) (docs.npmjs.com) (see: `npm ci`, `npm config`, `npm prune`, `npm dedupe`, and anything else that sounds interesting!)
