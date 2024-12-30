+++
title = 'Self-Hosted eBook Servers'
date = 2024-12-30T15:57:41-05:00
draft = true
+++

# Self-Hosted eBook Servers

## The Problem

Recently, I've been trying to get back into reading more actual books as an alternative to homelabbing, or watching more TV. It used to be that I had a great love for fantasy novels - and really, is there any greater form of escape?

On that note, I started looking into a self-hosted solution to remove my reliance on Amazon Kindle.

## Exploring Solutions

My requirements, I thought, were simple:

1. Self-hosted, open source server
2. Able to read on any device (I use a mix of Linux, Windows and Android devices.)
3. Tracking progress no matter which device I was on, so I can just pick up and read with whatever device I happen to have available.

My first thought of course, was Calibre - which has been the name in eBook management software for as long as I can remember.

I attempted to set it up on one of my local Docker host VMs and immediately was beset with issues.

This application clearly was never designed to work in the way that I was attempting to:

- Keep the database on the local Docker host to avoid any file-locking concerns
- Store the books themselves on my NAS, where they are protected by ZFS and backed up nightly.

Instead, you are basically forced to keep your `metadata.db` and other directories the same directory as the actual content.

Calibre also didn't appreciate trying to run the database over NFS - I had a host of permissions issues trying to do so. This was probably solvable, but would have required even more time than I'd already spent on this to run the application in a way that risked corrupting the database. Eventually I gave up on this, after hours of chasing down various Reddit posts and forum threads.

Instead, I set up Calibre directly on the NAS using the official [TrueNAS Scale Apps](https://www.truenas.com/apps/) for Calibre and Calibre-Web.

Finally, something that worked. I was able to import all of my books and connect to the content server over OPDS.

That was when I ran into another hiccup - apparently OPDS does not have any method of reporting book progress back to the Calibre server. The creator of Calibre suggests [using the browser viewer](https://www.mobileread.com/forums/showthread.php?t=296205#2), which is not great as far as user experiences go.

Reading via [Calibre-Web](https://github.com/janeczku/calibre-web) has the same issue - it doesn't update the reading position in Calibre itself. Ugh.

Next, I turned to the excellent [Awesome Selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted#document-management---e-books) list to see if there are any decent alternatives.

## Interim Solution

What I've wound up settling on for the moment is [Kavita](https://www.kavitareader.com/).

It also supports OPDS - but has the same limitations with every Android client that I've tried. However, the web interface is much more acceptable than the one provided with Calibre, even on mobile.

I'm still not exactly thrilled with this solution - I would much prefer a browser experience when reading on desktop, and using something like [Moon+ Reader](https://www.moondownload.com/) on Android, especially on my Galaxy Tab S9.

Curious if anyone has any other suggestions or anecdotes on how they have a similar environment configured!
