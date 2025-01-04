+++
title = 'Thoughts on starting a media server in 2025'
date = 2025-01-03T17:56:14-05:00
draft = true
+++

# Thoughts on starting a media server in 2025

Recently, someone on one of the less technical Discord servers that I frequent asked a question about setting up [Plex](https://plex.tv).

While Plex is still a perfectly capable media server - I still run it myself, simply because migrating the 20+ users I have using it to [Jellyfin](https://jellyfin.org) would be difficult - there are some [privacy concerns](https://www.reddit.com/r/selfhosted/comments/180maoe/plex_crossed_a_line_with_your_week_in_review/) over the changes that Plex has made in recent years.

This got me thinking, how would *I* suggest setting up a media server in 2025? What would that look like if someone was starting from nothing?

This article is oriented at newer users who have little knowledge of networking, but want to break free from the tyranny that is their cable provider or the various streaming services.

I used to recommend Alex Kretzschmar's [PerfectMediaServer](https://perfectmediaserver.com) website to people who are getting started, but that's gotten a bit too complicated, in my opinion. (Sorry Alex, but I don't think I want to push Linux newbies towards NixOS.)

## 1. Which media server do I want to use?

If you don't have interest in sharing your media outside of your home, go with Jellyfin.

It's a newer piece of software, but it is completely free and open source, and hardware transcoding is not locked behind a paywall like it is with Plex.

Another potential option is [Emby](https://emby.media), but while I've heard good things about it (Jellyfin happens to be a fork of it!) I have very little experience with it personally.

## 2. What hardware do I need?

The hardware requirements for running a basic, single person media server are more modest than you might expect. What I would recommend for someone getting started is a PC with a relatively recent Intel processor, 8GB of RAM, and an SSD boot drive. Something like an off-lease Dell Optiplex 3060 micro PC would work fine.

The reason for this?

**QuickSync.**

Intel QuickSync is a hardware transcoding technology integrated into the iGPU on modern Intel CPUs. This means that you don't need a separate video card for handling transcoding tasks between different video formats.

Transcoding is important for media servers, as they often need to transcode one format to another on the fly. Without hardware transcoding, you'll be stuck with software transcoding, which will result in a ton of unnecessary CPU usage and power draw.

It is by far the best in class when it comes to price to performance and performance per watt.

That said, avoid the earlier iterations of QuickSync, as they were far less performant.

Anything above 8th generation Intel should be sufficient for this task. Think i5-8500, i7-8700 - even the i3 variant would probably be enough for a low power machine.

You'll also need some storage. This doesn't have to be anything special - if your data needs are modest, you can probably get away with a single hard drive, or even a USB external hard drive, if that's all you have.

If you need more space, though, look into getting a NAS to store the media. This will be a separate box that your media server will talk to over the network in order to get to the media files.

There are pre-assembled solutions such as the [TrueNAS Mini](https://www.truenas.com/truenas-mini/), or [Synology](https://www.synology.com/en-us). However, these almost always come without hard drives that you will have provide yourself, and the units themselves are very expensive for the hardware that you're getting.

If you'd prefer to go the DIY route - I recommend having a look at [Brian C. Moses' blog](https://blog.briancmoses.com/categories/diy-nas/). He has some excellent guides for setting up affordable and power efficient systems that don't compromise too much on performance.

In case you were wondering - yes - you *can* combine both of these devices into one, and run your media server on your NAS. Just make sure you adjust your hardware requirements accordingly.

**You will need a more powerful processor and more RAM to run both of these services in one box.**

I would recommend at *minimum* an i5 processor, 8th gen or better, and 16GB of RAM. 32GB of RAM would be preferable.

## 3. What operating system should I run this on?

I know it's a difficult thing if all you've ever touched is Windows or Mac, but I don't recommend running your media server on either of those platforms.

You could run the media server on either Unraid or TrueNAS Scale - nothing wrong with either of those platforms, but I do recommend taking the time to learn how to install a Linux operating system like [Ubuntu](https://ubuntu.com/download/desktop) and a containerization stack like [Docker](https://www.docker.com).

{{% callout note %}}
I deliberately put the link for the Ubuntu Desktop installer there, as it will be less scary than the server operating systems I normally use, which are CLI only, without a graphical user interface.
{{% /callout %}}

Docker will give you *far* more flexibility when you're running it on a general purpose Linux OS rather than a specialized one like TrueNAS or Unraid.

But, it's all about what you're comfortable with. If you want to get started with your media server while running everything on Windows - go for it!

## 4. Is there any other software that I should use?

Oh boy. I'm so glad you asked.

This is a rabbit hole beyond imagining - once you start, I dare you to stop until *everything* with regards to your media collection is automated.

I will not go into more detail here, because it will make this post a *mile* long if I do. Maybe I'll make another post at some point going into detail about how I, in particular, have my media stack configured. **(Let me know in the comments if you're interested! Hint hint!)**

In no particular order:

- [Sonarr](https://sonarr.tv/) - TV show collection organizer / downloader
- [Radarr](https://radarr.video/) - Movies collection organizer / downloader
- [Lidarr](https://lidarr.audio/) - Music collection manager / downloader
- [Prowlarr](https://prowlarr.com) - Indexer manager for the other *Arr apps.
- [Bazarr](https://www.bazarr.media/) - Subtitles manager
- [Overseerr](https://overseerr.dev/) / [Jellyseerr](https://github.com/Fallenbagel/jellyseerr) - Manage requests for movies and TV shows.
- [Recylarr](https://recyclarr.dev/wiki/getting-started/) - Automatically applies the [TraSH Guides](https://trash-guides.info) recommendations to your media. (I'll admit this one is a bit niche, but I love it.)
- [Sabnzbd](https://sabnzbd.org/) - Usenet client

{{% callout note %}}
(Jellyseerr being a fork of Overseerr that works with Jellyfin. Supposedly, Jellyfin support is coming to Overseerr, but it hasn't made it there yet.)
{{% /callout %}}

## 5. Final Thoughts

Some final notes before I wrap this up.

Please, if you care about your own sanity - organize your media in a way that makes sense.

Make a central directory on whatever you're using to store everything and call it whatever.

`/mnt/media`, `D:\media`, `/Volumes/Rick Astley Fan Page`, I don't care.

I *highly* recommend structuring it the way they suggest to in the aforementioned [TraSH Guides](https://trash-guides.info) as it will save you a lot of headaches down the line.

It all comes back to this - you have a *central* folder, whatever it is called, and all of your media goes *inside it*, and is broken down further from there.

This allows you to have *one* folder you need to share out rather than twenty, and in most cases will allow you to use [hard links](https://trash-guides.info/File-and-Folder-Structure/Hardlinks-and-Instant-Moves/#what-are-hardlinks).

This also leads me into something else that is very important that will be made easier by having everything centralized into one root directory.

# Backups.
{style=color:red}

I don't care if you have hardware RAID, or ZFS, btrfs, whatever. I don't care what level of redundancy you have.

[RAID is not a backup!](https://www.raidisnotabackup.com/)

If you don't care if you lose your media collection, that's fine. But if you're like me, you've been collecting and curating this stuff for *years* and losing it would hurt. So, I have backups.

Even if your backup is just copying one external hard drive to another periodically - that's better than nothing.

I'm of the opinion that backups should be automated, and something else automated should be checking on them periodically to make sure that they're doing what they should be doing.

But please, however you have to do it - make backups!
