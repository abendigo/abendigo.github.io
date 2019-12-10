---
layout: post
title: Github Actions
comments: true
---

About a month ago, [GitHub][github] made [GitHub Actions][actions] generally available. Earlier
in the year, they also made the [GitHub Package Repositry][packages] generally available.
This was the exact combination that I needed for a couple personal projects, so I immediately
migrated my builds from [CircleCI][circleci].

There are a ton of blog posts and tutorials about setting up this kind of stuff, so after a couple
days experimenting and googleing, here is my `ci.yml`:

    name: Build, Tag, and Publish Docker image
    env:
      REGISTRY: docker.pkg.github.com
      IMAGE: auth

    on:
      push:
        branches:
          - master

    jobs:
      build:
        name: Build, Tag, and Publish Docker image
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v1
          - name: Generate VERSION
            run: echo "::set-env name=VERSION::$(git describe --tags --always --dirty)"
          - name: Build IMAGE
            run: docker build -t ${IMAGE} .
          - name: Tag IMAGE:VERSION and IMAGE:latest
            run: |
              docker tag ${IMAGE} ${REGISTRY}/${{ github.repository }}/${IMAGE}:${VERSION}
              docker tag ${IMAGE} ${REGISTRY}/${{ github.repository }}/${IMAGE}:latest
          - name: Login to Registry
            run: echo ${{ secrets.GITHUB_DOCKER_PASSWORD }} | docker login ${REGISTRY} -u ${{ secrets.GITHUB_DOCKER_USERNAME }} --password-stdin
          - name: Publish IMAGE:VERSION and IMAGE:latest
            run: |
              docker push ${REGISTRY}/${{ github.repository }}/${IMAGE}:${VERSION}
              docker push ${REGISTRY}/${{ github.repository }}/${IMAGE}:latest

Now, every time I push to master, the code is checked out, a docker image is created, and saved to
the [Package Repository][auth].


[github]: https://github.com
[actions]: https://github.com/features/actions
[packages]: https://github.com/features/packages
[circleci]: https://circleci.com/
[auth]: https://github.com/abendigo/auth/packages
