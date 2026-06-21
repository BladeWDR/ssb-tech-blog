+++
title = 'Fluxer Over Tailscale'
date = 2026-06-20T21:36:21-04:00
draft = false
+++

This weekend I've been playing with setting up a self-hosted [Fluxer](https://fluxer.app) instance.

In my traditional fashion, I made things much harder by myself by refusing to use their provided Caddy setup, and instead using Traefik.

And for an additional challenge - I wanted this to be completely inaccessible from the public Internet, instead using my Tailscale mesh network.

The setup required some heavy modifications to the Docker Compose file provided by Fluxer's team, and a few tweaks to `livekit.yaml`.

{{% callout note %}}
The Docker Compose file for this service is extremely long, so instead of posting it inline here as I normally do, I have created this [Gist](https://gist.github.com/BladeWDR/1b062af5eede7e0c46a3446d46b4f889) containing both the modified Compose project and `livekit.yaml`.
{{% /callout %}}

With this setup, I'm able to access Fluxer at `fluxer.mytailnet.ts.net` with full TLS encryption and valid certificates.

I am using:

- A Tailscale sidecar container
- The Tailscale Traefik provider for TLS certificates

Effectively, what you need to do is tell Traefik and the Livekit service to share the Tailscale container's network namespace.

Then we tell Livekit to advertise the Tailscale container's IP address instead of the one for the Docker bridge (see `livekit.yaml`, it's the `node_ip` setting).

Note - you'll need to add the `TS_SOCKET: /var/run/tailscale/tailscaled.sock` environment variable to the Tailscale container, and pass the Tailscale socket through to the Traefik container - it needs access to the Tailscale daemon to be able to grab your certificates.

You'll also need to have [enabled HTTPS on your Tailnet](https://tailscale.com/docs/how-to/set-up-https-certificates).

**What works**

- Text chat
- Voice chat

**What doesn't work (yet)**

- Video chat - I think this has something to do with Livekit video codecs. I need to play a bit more with this and do some more testing.
