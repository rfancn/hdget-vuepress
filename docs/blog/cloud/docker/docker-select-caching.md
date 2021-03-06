---
title: Selectively disable caching for specific RUN commands in Dockerfile
date: '2017-12-27'
draft: false
categories:
- 博客
tags:
- docker
- cloud
---
If you using Dockerfile, you should know about caching for faster builds. Sometimes, you need to rebuild without cache. you can run docker build with `--no-cache` option that will disable all layer cache.

There is problem when you should use some Dockerfile commands like ADD or COPY to copy file form host to container.

You need to copy new file to container then rebuild again without change any line of Dockerfile but the new file not was copy to container. The last build using layer cache that contain old file instead of rebuild with new file. So this problem can solve easily by using --no-cache buuild option that disable all layer cache.

Sometimes, you need to rebuild only some layer not all layer. For this case, there is not native solution for Docker but there is some trick about ARG command in Dockerfile and `--build-arg` build option that I found on New feature request: Selectively disable caching for specific RUN commands in Dockerfile for more detail you should look at solution description and Dockerfile - ARG

add ARG command to your Dockerfile

    # Dockerfile
    # add this and below command will run without cache
    ARG CACHEBUST=1

when you need to rebuild with selected cache, run it with --build-arg option

    $ docker build -t your-image --build-arg CACHEBUST=$(date +%s) .

then only layer below ARG command in Dockerfile will rebuild.

#### References:
- [Build Cache - Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache)
- [Dockerfile - ARG](https://docs.docker.com/engine/reference/builder/#arg)
