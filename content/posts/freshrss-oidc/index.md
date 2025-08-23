+++
title = 'Connecting an Existing FreshRSS instance to OIDC'
date = 2025-08-15T21:05:44-04:00
draft = false
+++

# Connecting an existing FreshRSS instance to OIDC

Specifically, how to do so without starting over.

<span style=color:red>**WARNING:** I am not responsible for you losing data by doing this. Make sure you have a recent backup before you go digging around and changing things in the filesystem.</span>

## Overview

It came to my attention yesterday that the [FreshRSS](https://freshrss.org/index.html) project directly supports OIDC now.

I've been using the Traefik [ForwardAuth middleware](https://doc.traefik.io/traefik/middlewares/http/forwardauth/) to get the same functionality for some time, but this allows me to get rid of a layer.

It was also slightly clunky, since after I'd logged in through Authentik, I'd have to also authenticate via FreshRSS. I'm aware I could have simply disabled authentication, but I chose not to at the time.

I had been using the [LinuxServer.io](https://linuxserver.io) Docker container, but this functionality depended on Apache's OIDC module, so it was unsupported. (The LinuxServer.io container uses nginx instead of Apache.)

## OIDC Setup

The first thing I needed to do was switch to the [official container on DockerHub](https://hub.docker.com/r/freshrss/freshrss).

The volume layout was a bit different from the LinuxServer container, so I had to stop the container and move some files around.

All you should need to do is move the `data` and `extensions` folders out from under the `config` directory that the LSIO container uses, and make them their own separate bind mounts.

Next, I set about configuring OIDC.

I followed their [official instructions](https://freshrss.github.io/FreshRSS/en/admins/16_OpenID-Connect-Authentik.html) to set up OIDC with [Authentik](https://goauthentik.io/).

This was _mostly_ fine, though I had to make a few tweaks to their environment variables. The `OIDC_SCOPES` variable in particular was my issue - it would not work if I had it in quotes. I can only assume that this was being interpreted as a single string by the application.

Here's what my Docker Compose file wound up looking like at the end. The OIDC values have been replaced with gibberish, make sure to substitute your own if you copy and paste.

{{% callout note %}}
2025/08/21 - After posting this I began noticing that my feeds were no longer auto-updating. As it turns out, the FreshRSS official Docker image [disabled this feature by default unless you pass it the CRON_MIN environment variable](https://freshrss.github.io/FreshRSS/en/admins/08_FeedUpdates.html). I have updated the Docker Compose file below with a setting that works.
{{% /callout %}}

```yaml
services:
  freshrss:
    container_name: freshrss
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
      - CRON_MIN=2,32
      - TRUSTED_PROXY=172.23.0.0/16
      - OIDC_ENABLED=1
      - OIDC_PROVIDER_METADATA_URL="https://auth.example.com/application/o/fresh-rss-oidc/.well-known/openid-configuration"
      - OIDC_REMOTE_USER_CLAIM=preferred_username
      - OIDC_CLIENT_ID=1234455262233
      - OIDC_CLIENT_SECRET=7dfadkfjakldgjad7897324
      - OIDC_CLIENT_CRYPTO_KEY=dfajfakjglkdg70708723434
      - OIDC_SCOPES=openid email profile
      - OIDC_X_FORWARDED_HEADERS=X-Forwarded-Host X-Forwarded-Port X-Forwarded-Proto
    volumes:
      - ./data:/var/www/FreshRSS/data
      - ./extensions:/var/www/FreshRSS/extensions
    restart: unless-stopped
    image: "freshrss/freshrss:latest"
    labels:
      - traefik.enable=true
      - traefik.http.routers.freshrss-external.rule=Host(`rss.example.com`)
      - traefik.http.routers.freshrss-external.entrypoints=https
      - traefik.http.routers.freshrss-external.middlewares=default-headers@file
      - traefik.http.routers.freshrss-external.tls=true
      - traefik.http.routers.freshrss-external.service=freshrss-external
      - traefik.http.services.freshrss-external.loadbalancer.server.port=80
    networks:
      - proxy

networks:
  proxy:
    external: true
```

I'm not 100% certain that the `TRUSTED_PROXY` setting is necessary. The subnet there is the subnet used by my `proxy` docker network. It works and I'm too lazy to try removing it.

You can get this subnet by running `docker network inspect <network-name>`

```shell
> docker-ubuntu in ~/docker/freshrss/data
docker network inspect proxy
docker network inspect proxy
[
    {
        "Name": "proxy",
        "Id": "4b6ea83029c955c6f7609af4552ed053760aadbb6de5c72f8390d3cfa96023b1",
        "Created": "2022-09-24T02:01:57.289063399Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.23.0.0/16", <-------- This
                    "Gateway": "172.23.0.1"
                }
            ]
        },

<snip>
```

## Not losing data

Onto the meat and potatoes.

In order to not log into a completely fresh profile with no articles and none of your settings - your user ID in FreshRSS _needs_ to match that of your identity provider.

Mine did not, so I had to do a little detective work.

I stopped the container with a `docker compose down` and started poking about in the filesystem, trying to figure out how I could change my username.

As it turns out, it's quite simple - each user has their own directory under `data/users/`, and each user has their own individual sqlite database. I poked around in the database a little, but there didn't seem to be any indication of who owned each entry it stored in its tables.

This led me to believe that maybe FreshRSS was just reading the directory structure to find its list of users.

```shell
docker-ubuntu in ~/docker/freshrss/data
> tree users
users
├── MyUsername
│   ├── config.php
│   ├── db.sqlite
│   └── log.txt
├── _
│   ├── db.sqlite
│   ├── index.html
│   ├── log.txt
│   ├── log_api.txt
│   └── log_pshb.txt
└── index.html

2 directories, 10 files
```

So as an experiment, I simply renamed the user folder under `data/users` to match the one from my identity provider and started FreshRSS back up. Imagine my surprise when that simply worked.

I was logged in automatically with Authentik SSO, and I had all of my feeds, saved articles and settings.
