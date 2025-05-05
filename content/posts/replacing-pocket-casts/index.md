+++
title = 'Replacing Pocket Casts'
date = 2025-05-05T08:53:26-04:00
draft = false
+++

# Replacing Pocket Casts

## Overview

I've been a user of [Pocket Casts](https://pocketcasts.com) for a long time now. There's nothing wrong with it, and in fact it's been a fantastic experience the entire time I've used it.

However, I've been trying to move whatever I can to open-source, self-hosted projects, and this was one that I'd wanted to take a crack at for quite a while.

Here were my requirements for a podcast manager:

1. Self-hosted and open-source.
2. The ability to listen on either mobile or desktop, and pick up from where I left off in either location.

I went through a few iterations of this, and I'm still not quite happy with the result, but let me go over the projects that I tried out, and the issues I ran into with them.

## PinePods

[PinePods](https://github.com/madeofpendletonwool/PinePods) is a newer project, built mostly with Rust and Python.

I tried this one first because it had a pretty web interface, a companion Android app, and support for sync through Gpodder compatible servers like [Opodsync](https://github.com/kd2org/opodsync) or the [NextCloud GpodderSync app.](https://apps.nextcloud.com/apps/gpoddersync).

Installation was fairly straightforward, and I chose to integrate it with my existing Authentik instance for OIDC sign on.

All of that worked great - until I installed the mobile app and realized that OIDC is not yet supported. I assume that this is the reason that I was not seeing my progress synced between the mobile app and the web interface.

Next I tried using [AntennaPod](https://antennapod.org/) for the mobile app, with GpodderSync on NextCloud as the backend.

Somehow enabling GpodderSync in PinePods wiped out all of my existing podcast subscriptions. Thankfully I had them all backed up in an OPML file still, so it was just a matter of re-importing the file again.

However, none of the podcasts I had added to PinePods ever showed up in GpodderSync on NextCloud. However, adding a podcast via AntennaPod worked fine.

Supposedly there is a large update to sync coming to PinePods in the next few days, but I chose to move on and try out something else.

## PodFetch

[PodFetch](https://github.com/SamTV12345/PodFetch) is another podcast manager. This one comes with its own built-in Gpodder sync mechanism that just needs to be enabled via an environment variable.

The good:

I actually think PodFetch has the nicer UI of the two.

The bad:

I found the documentation for this project severely lacking and often found myself digging through GitHub issues to try and figure out how to configure certain things correctly.

Once again, OIDC is essentially a non-starter unless you're not planning on ever using a mobile app.

Also, make sure that the user you try to connect AntennaPod with is _not_ an administrator, or it will just fail. I had to enable additional logging in PodFetch to figure that out.

Once I finally figured out that headache, I discovered that sync is essentially broken. Listening to a podcast on either end and then forcing a full sync in AntennaPod does not change either side. I'm not sure what's going on here, but this was the nail in the coffin. I was no longer interested in spending more time trying to get this working.

## NextPod

Next, I tried out [NextPod](https://apps.nextcloud.com/apps/nextpod). This is the companion app to GpodderSync.

It is extremely basic and missing what I'd consider basic features like skip forward and back buttons, but at the very least - the synchronization of listen state _works reliably_.

For the moment I've settled on this setup:

- GpodderSync for the backend, which keeps track of my subscriptions and listen state
- NextPod for listening on desktop
- AntennaPod for listening on mobile

It does essentially everything that was on my laundry list, but I'm not particularly happy with NextPod as my desktop solution. It works, but it lacks even the most basic management features, and the playback UI is, as I said, barebones.

If anyone else has any suggestions I'd like to hear about them. I'll check out the updated PinePods when the sync updates drops, but for the moment I'll leave this as it is.
