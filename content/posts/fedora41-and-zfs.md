+++
date = '2024-12-09T20:40:18-05:00'
draft = false
title = 'Fedora 41 and ZFS'
+++

{{% callout note %}}
ZFS 2.2.7 has been released with up to Linux kernel 6.12 support, so this workaround is no longer necessary.
I'll leave the post up in case this is ever useful in the future.
{{% /callout %}}

I previously posted about how to [build a desktop OS from the Fedora Server installer]({{< relref "posts/building-fedora-server-into-a-desktop/index.md" >}}).

This is a bit of a follow up to that post.

I've now been running Fedora 41 for a few days and it's been a mostly pleasant experience.

However, I am also an avid ZFS user, and the current stable release of OpenZFS [does not support Linux kernel 6.11 yet](https://github.com/openzfs/zfs/issues/16590).

So, being the enterprising fellow that I am, I chose to build ZFS 2.3.0 RC3 from source using the [official instructions](https://openzfs.github.io/openzfs-docs/Developer%20Resources/Building%20ZFS.html)

That went well, but one thing that tripped me up (and caused some issues for specific applications like syncthing, which keeps it's local database in the user's home directory) is that when you do this, unlike when you install `zfsutils-linux` on Ubuntu, it doesn't set up the automatic import and mounting of your pool.

It took some digging around to find the correct process, because it seems to be a bit different between the official ZFS source and the one that ships with Ubuntu, but here's what wound up working for me:

First, go through and enable the following services:

```
sudo systemctl enable zfs.target zfs-mount.target zfs-import.target zfs-import-cache.service
```

There are plenty of posts online telling you that you should create your cache file in `/etc/zfs/zfs-list-cache/`, but this is wrong (or at least outdated. I’m not sure if that location is Ubuntu specific?)

The ACTUAL location of the cache file (which you can see if you actually read through the systemd unit file for zfs-import-cache.service) is `/usr/local/etc/zfs`.

If the file doesn't exist, create it.

```
sudo touch /usr/local/etc/zfs/zpool.cache
```

Import your pool.

```
sudo zpool import poolname
```

If you print out the cache file now it should no longer be empty.

```
cat /usr/local/etc/zfs/zpool.cache
```

Next time you reboot your pools should automatically mount and mount all filesystems.

And yes, I did cross post this over on the [PracticalZFS forum.](https://discourse.practicalzfs.com/t/psa-zfs-automount-when-building-from-source/1970/1)
