+++
title = 'Wireguard configuration concepts'
date = 2025-10-26T15:04:55-04:00
draft = false
+++

# Wireguard concepts

## The point of this post

This is intended as a high level starting point for someone who is new to using Wireguard.

If you spot any errors or inaccuracies, feel free to open a pull request in the [Github repo](https://github.com/bladewdr/ssb-tech-blog), or leave a comment.

## Overview

[Wireguard](https://www.wireguard.com/) is a modern VPN protocol.

Instead of using a traditional client server model, Wireguard uses a peer-to-peer mechanism. Authentication is handled with private and public key pairs, and optionally a pre-shared key.

## Wireguard configuration on Linux

There are many ways to configure Wireguard on Linux, but the one that I typically use is by setting up a file in `/etc/wireguard`.

On Linux, this is the default location that the `wg-quick` userspace utility looks for Wireguard interface configurations.

The interface configuration can contain both the definition for the interface as well as any peers.

### Anatomy of the Wireguard configuration file

Consider the following configuration file: `/etc/wireguard/demo.conf`.

When using `wg-quick`, the interface name will be generated from the name of the configuration file. In this case, the interface name would be `demo`.

Sample configuration file:

```
[Interface]

Address = 192.168.99.5/24
PrivateKey = 2COm4EMxP5tcF9cpgCxBAsSGRwrjoMmWjkr+emGtylY=
ListenPort = 51820

[Peer]

PublicKey = Tlm3payEVQWCj7GK7Y7usejF4mix5FBULZryxfu9kxY=
PreSharedKey = chooseabetterkey
AllowedIPs = 192.168.99.8/32,10.10.33.0/24
Endpoint = demo.wireguard.com:51820
```

Lets break down this config file into its components.

Wireguard uses an INI-like syntax.

There are 2 top level blocks that can be configured, `[Interface]` and `[Peer]`. There can be multiple `[Peer]` definitions, but only one `[Interface]`.

#### The Interface Block

**Address**
: What address do you want this particular peer to have? Make sure you set the correct CIDR notation for your subnet size. This one is `/24` for a 254 address subnet.

**PrivateKey**
: The private key is a secret! I've generated this one just for this demo, it will be discarded after.

: But make sure you secure these properly!

**ListenPort**
: The default listen port is 51820, but you can set it to anything you want. This is the network port that Wireguard will use to listen on this interface.

#### The Peer Block

**PublicKey**
: This is the public key associated with the _peer's_ private key, _not_ the private key of this interface.

**PreSharedKey**
: Preshared keys are optional but good for additional security. This is a key that needs to match on both sides of the connection.

**AllowedIPs**
: The AllowedIPs stanza is an interesting one. It is what Wireguard uses to make internal routing decisions, but those packets need to arrive at the Wireguard interface first.

: This is the reason I like to use wg-quick to set up my interfaces - if you don't, you'll need to add the routes yourself. This is personal preference.

: The configuration above would allow this peer to have the 192.168.99.8 IP on the Wireguard internal subnet, and any traffic destined for 10.10.33.0/24 would also get passed along.

: Note how I'm using `/32` for the Wireguard internal subnet here. This is because I only want that _one_ IP to be valid for this peer. The subnet is still a `/24` as configured on the interface itself.

**Endpoint**
: Endpoint is technically an optional field.

: But it does need to be set on at least one side of each connection.

: At least one of the peers needs to have a public IP address that's reachable on the internet. (And the `ListenPort` needs to be open.)

There are other stanzas that can be used in the configuration file, but these are the basic ones.

You can see the [wg](https://www.man7.org/linux/man-pages/man8/wg.8.html) and [wg-quick](https://www.man7.org/linux/man-pages/man8/wg-quick.8.html) man pages for more information on configuration file syntax.

### Using wg-quick with systemd

When installing Wireguard on Linux, it comes with a template [systemd](https://en.wikipedia.org/wiki/Systemd) unit file for `wg-quick`.

This is a wrapper that allows you to quickly set Wireguard configurations that load at startup.

To start up our config file from earlier using wg-quick, you can do it like so:

```bash
sudo systemctl start wg-quick@demo
```

Where the text after the `@` symbol will always been the config file name without its `.conf` extension. For more info on this, see the [Arch Wiki page on systemd](https://wiki.archlinux.org/title/Systemd#Using_units).

If you want to make the configuration start at boot, you can enable the systemd unit.

```bash
sudo systemctl enable wg-quick@demo
```

{{% callout note %}}
Yes, you can have multiple Wireguard interfaces on one machine. Just make sure that they listen on different ports.
{{% /callout %}}

## General usage notes

While Wireguard as a protocol is peer-to-peer, many people do still use it in a client-server architecture.

One use case for this is if you're using Wireguard to set up your own privacy VPN endpoint.

{{% callout note %}}
If you do use Wireguard as a VPN server, remember that you also need to enable [IP forwarding](https://askubuntu.com/questions/311053/how-to-make-ip-forwarding-permanent).
{{% /callout %}}
