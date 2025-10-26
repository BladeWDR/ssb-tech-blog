+++
title = 'Getting started with Docker'
date = 2025-10-26T15:41:37-04:00
draft = false
+++

# Getting started with Docker

_So, you wanna use docker-compose..._

## Introduction

Welcome! If you're looking at this post, I assume that you're new to Docker and you want to learn to get up and running quickly.

There are several areas of Docker that confuse new users, and I will do my best to address most of the most common pitfalls in this post.

## docker run vs docker compose

Don't use `docker run`. If the thing you want to run in Docker only gives you the `docker run` syntax, you take that thing over to [Composerize](https://www.composerize.com/) and you convert it.

There's nothing particularly _wrong_ with `docker run`, but `docker compose` is far easier to maintain and troubleshoot.

Tools like [yamllint](https://github.com/adrienverge/yamllint) make diagnosing issues with compose file formatting much more tractable.

## Docker networking

This seems to be the area that causes the most confusion.

Don't worry, you don't need to be a network engineer to get a grasp of this, but having a good understanding of how networking works is very helpful.

### Network namespaces

Docker uses Linux kernel [network namespaces](https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/) to achieve isolation. You don't need to really understand any of this, but click the link if you're at all interested.

Effectively what this means is that containers by default are isolated from both the host operating system's networking stack, and from each other.

Whenever you create a new Docker compose project, by default it will create a new Docker network bridge, with its own internal subnet and DNS.

**_Containers in the same Docker compose project (in the same docker-compose.yml file, or separate files linked together with the `-p` flag) share the same network bridge, and thereby are able to communicate with each other._**

### Communicating with containers by name

Docker also does some things under the hood to make it possible for docker containers in the same stack to communicate with each other by service name.

To illustrate this, lets go with a simple example.

```yaml
services:
  app:
    image: myapp
    restart: unless-stopped
    environment:
      - POSTGRES_CONNECTION_STRING=postgresql://postgres-db1:5432
    volumes:
      - ./config:/config
  postgres-db1:
    image: postgres
    restart: unless-stopped
```

Above, we have an extremely simple docker compose file, with a single application and a PostgreSQL database.

You can see in the `POSTGRES_CONNECTION_STRING` environment variable that it is using the name of the service to communicate with the database.

### Exposing ports

Next, lets get into how Docker handles exposing ports.

Lets take the Jellyfin docker compose file for an example:

```yaml
---
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - JELLYFIN_PublishedServerUrl=http://192.168.0.5 #optional
    volumes:
      - /path/to/jellyfin/library:/config
      - /path/to/tvseries:/data/tvshows
      - /path/to/movies:/data/movies
    ports:
      - 8096:8096
      - 8920:8920 #optional
      - 7359:7359/udp #optional
      - 1900:1900/udp #optional
    restart: unless-stopped
```

You'll notice the `ports` key. These are what's known as [published ports](https://docs.docker.com/get-started/docker-concepts/running-containers/publishing-ports/).

The number on the left is what is mapped _outside_ the container (on the machine running Docker), and the one on the right is inside the container.

In 99% of cases, you want to leave the number on the right alone.

For example, if I wanted the Jellyfin web interface, which by default runs on port 8096, and expose it on port 9001, I could change the compose file like so:

```yaml
ports:
  - 9001:8096
  - 8920:8920 #optional
  - 7359:7359/udp #optional
  - 1900:1900/udp #optional
```

Now, on to another example.

In many cases, you may not want to expose any ports at all!

If users external to the Docker host don't need to be able to touch that port, then simply don't publish it.

Take this other example:

```yaml
services:
  proxy:
    image: "jc21/nginx-proxy-manager:latest"
    restart: unless-stopped
    ports:
      - "80:80" # Public HTTP Port
      - "443:443" # Public HTTPS Port
      - "81:81" # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      DOMAIN: "https://vw.domain.tld"
    volumes:
      - ./vw-data/:/data/
```

You'll notice that the [Vaultwarden](https://github.com/dani-garcia/vaultwarden) container doesn't have any published ports.

That's because it doesn't need them. In this scenario, we would go into [Nginx Proxy Manager](https://nginxproxymanager.com/) and configure a proxy host for it.

All communication with Vaultwarden would go through the proxy, so there's no need to expose the port.

**_Remember that when containers are talking internally like this, you use the INTERNAL port (the one that would be on the right), instead of the one you would have exposed on the outside._**

Nginx can talk to Vaultwarden and vice versa - there is no reason for a user to directly interact with the other container.

This is also valid syntax:

```yaml
ports:
  - 127.0.0.1:8000:80
```

This is telling the container to only publish port 8000 on `localhost`. You can substitute any other IP here to make it only listen on that one interface.

I don't use this a whole lot.

## Docker volumes

Docker volumes are simply a piece of your filesystem that you're giving to the container to store persistent data.

By definition, Docker containers are [ephemeral.](https://www.merriam-webster.com/dictionary/ephemeral)

Unless you configure a volume, any data produced while the container is running will be lost when the container is destroyed.

The two most popular ways to configure volumes in Docker are named Docker volumes and bind mounts.

This is a named docker volume:

```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
```

And this is a bind mount:

```yaml
volumes:
  - /apps/myapp/postgres-data:/var/lib/postgresql/data
```

{{% callout note %}}
If it's a filesystem path on the left, it's a bind mount.

The path on the right is the [mountpoint](https://askubuntu.com/questions/83518/what-are-mountpoints-and-which-one-should-i-pick) inside the container.
{{% /callout %}}

Honestly as far as I'm concerned it's mostly personal preference which you use. I like using bind mounts as it makes it easier to keep track of where your data is.

Named docker volumes will store the data in a subdirectory under `/var/lib/docker/volumes/`.

Whichever one you use, please make sure you do backups... I'm not going to help you try and get your data back, sorry.
