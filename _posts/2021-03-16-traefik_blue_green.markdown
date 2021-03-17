---
layout: post
title: Simple Blue/Green Deployments with Traefik and Docker
comments: true
---

I have been using [Traefik][traefik] with [Docker][docker] for a while
to host some personal projects, and to experiment with [devops][devops].

One thing I have always wanted to accomplish is full [Continuous Deployment][cd],
with zero downtime, using [Blue-Green Deployments][blue-green-deployments],
but I didn't want all the extra overhead of setting up [Swarm][swarm] or [Kubernetes][kubernetes].

Here's how I recently accomplished this goal.

First, create a [traefik.yml][traefik.yml] file for static configuration:

    entryPoints:
      http:
        address: ":80"
      https:
        address: ":443"

    providers:
      file:
        directory: /etc/traefik/dynamic
        watch: true

    api:
      dashboard: true
      insecure: true

Then create a directory, `dynamic` for the dynamic configurations. There is nothing in it yet.

Next create a [docker-compose.yml][docker-compose.yml] file to spin up Traefik and hook up
some volumes:

    services:
      traefik:
        image: traefik:v2.4
        ports:
          - 80:80
          - 443:443
          - 8080:8080
        volumes:
          - ./traefik.yml:/etc/traefik/traefik.yml
          - ./dynamic:/etc/traefik/dynamic:ro

Start it up, `docker-compose up -d`, and vist [http://localhost:8080][localhost].
If everything worked, you should see the Traefik dashboard!

Now we need our `blue` and `green` services.  Created two simple html files, `blue.html`

    <h2>Blue<h2>

and `green.html`.

    <h2>Green<h2>

Then, create two simple [Dockerfiles][dockerfile], `Dockerfile.blue`

    FROM nginx
    COPY blue.html /usr/share/nginx/html/index.html

and `Dockerfile.green`.

    FROM nginx
    COPY green.html /usr/share/nginx/html/index.html

Build and label the images:

    docker build -t blue -f Dockerfile.blue .
    docker build -t green -f Dockerfile.green .

Let's test them out:

    docker run -p 8081:80 --rm blue
    docker run -p 8082:80 --rm green

Then, vist [http://localhost:8081][blue.localhost] and [http://localhost:8082][green.localhost].
Nice! Ok, lets shut down those containers, and update our docker-compose.yml file.
Add the following:

    blue:
      image: blue

    green:
      image: green

Then start them, `docker-compose up -d`. If you run `docker ps` or `docker-compose ps`
you should now see 3 containers running.

We need to tell Traefik about these services, so in the `dynamic` directory, we are going to add 4 files,
`http.services.blue.yml`:

    http:
      services:
        blue:
          loadBalancer:
            servers:
              - url: 'http://blue/'

`http.services.green.yml`:

    http:
      services:
        green:
          loadBalancer:
            servers:
              - url: 'http://green/'

`http.routers.blue-docker-localhost.yml`:

    http:
      routers:
        blue-docker-localhost:
          rule: Host(`blue.docker.localhost`)
          service: blue@file

`http.routers.green-docker-localhost.yml`:

    http:
      routers:
        green-docker-localhost:
          rule: Host(`green.docker.localhost`)
          service: blue@file

Load the [Traefik Dashboard][dashboard], and you should see the routes and services defined.
Vist [blue.docker-localhost][blue.docker.localhost] and [green.docker.localhost][green.docker.localhost].

Almost there! Just one final step. Add the following file
`http.routers.docker-localhost.yml`:

    http:
      routers:
        docker-localhost:
          rule: Host(`docker.localhost`)
          service: blue@file

See that last line? Thats where you decide which service, `blue` or `green`, maps to
[http://docker.localhost][docker.localhost].
When you want to change it, just change this one value. Traefik watches for changes in this directory, and updates
almost immediatly.

In a followup post, I will build a simple app that shows you which service is selected, and lets you change it.

[traefik]: https://traefik.io/traefik/
[docker]: https://www.docker.com/products/docker-desktop
[devops]: https://en.wikipedia.org/wiki/DevOps
[cd]: https://en.wikipedia.org/wiki/Continuous_deployment
[blue-green-deployments]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[swarm]: https://docs.docker.com/engine/swarm/
[kubernetes]: https://kubernetes.io/
[traefik.yml]: https://doc.traefik.io/traefik/getting-started/configuration-overview/
[docker-compose.yml]: https://docs.docker.com/compose/compose-file/
[localhost]: http://localhost:8080
[dockerfile]: https://docs.docker.com/engine/reference/builder/
[blue.localhost]: http://localhost:8081
[green.localhost]: http://localhost:8081
[dashboard]: http://localhost:8080/dashboard/#/
[blue.docker.localhost]: http://blue.docker.localhost
[green.docker.localhost]: http://green.docker.localhost
[docker.localhost]: http://docker.localhost
