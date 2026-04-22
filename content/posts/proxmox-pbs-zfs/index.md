+++
title = 'Deduplicated Proxmox backups with Proxmox Backup Server and ZFS'
date = 2026-04-22T23:23:10Z
draft = true
+++

## Overview

There are many ways to back up your [Proxmox](https://proxmox.com/en/products/proxmox-virtual-environment/overview) virtual machines.

One of the most popular ways to do so is via [Proxmox Backup Server](https://proxmox.com/en/products/proxmox-backup-server/overview), which gives you many advantages:

- Deduplication of common data.
- The ability to cherry pick files out of your backups.
- Tight integration with the Proxmox ecosystem.

However, how is one to achieve proper 3-2-1 backups using this mechanism?

One way is to run multiple datastores, some local and some using Amazon S3 compatible endpoints.

However, this is a bit much for me, so I came up with a simpler method.

## The Details

I run my PBS on a [Beelink N100 mini PC](https://www.amazon.com/dp/B0BVFKN7ZL), in which I've installed a SATA SSD that I formatted with a zpool.

I then installed Jim Salter's excellent [Sanoid utility](https://github.com/jimsalterjrs/sanoid), configuring snapshots on the PBS server.

Then, once I had my initial snapshots, I went to my TrueNAS backup server and created a new ZFS replication task.

I used this to _pull_ the snapshots for the datastore to the backup server.

This allows me to maintain the deduplicated nature of the Proxmox Backup Server datastore.

If I ever have a failure of the Proxmox Backup Server, I can simply set up a new zpool on the repaired system and reverse the direction of the replication.
