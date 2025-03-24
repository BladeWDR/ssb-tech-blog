+++
title = 'TrueNAS Scale - ZFS Replication Without Root'
date = 2025-03-21T15:15:20-04:00
draft = false
+++

## Overview

ZFS replication is my favored way of doing backups. In my homelab I have 2 systems running TrueNAS scale - one on my production hypervisor, with the disk controller passed through, and the backup server, which is just a baremetal TrueNAS Scale installation.

Unfortunately, most of the tutorials on this process seem to use either the root account or at the very least, an account with admin privileges. In this article, I'm going to go over how you can get a ZFS replication setup working with a completely unprivileged user, without ever leaving the GUI.

I will be doing a *pull replication* - where the production system has NO access to the backup system. The backup system will reach out and initiate the backups.

That way, there's no risk of the backup server being compromised because you had SSH credentials for it just lying around on production.

You should also lock down access to the backup system in other ways, but that's beyond the scope of this article.

## Create user accounts for backup

Go to Credentials > Users and create new users. Just create normal user accounts. Don't give it any extra permissions. I also recommend *unchecking* the box that allows it to be used for SMB authentication.

Make sure that you change the login shell from "nologin" to "bash" or "zsh".

I normally just call mine backup_user, but you can make it anything you want.

## Create SSH credentials

### On the backup system:

1. Click Credentials > Backup Credentials
2. Add an SSH Keypair. Give it a name. Click on generate keypair.
    - This will generate a PUBLIC key and a PRIVATE key. The private key will be used on the backup system to verify the public key.
    - I recommend copying the public key into a notepad or something. We'll need it again in a moment.
    - Click Save.
3. Add a new SSH Connection. Name it something that makes sense.
4. Setup method - manual
5. For host, enter the IP address of the production system. If your machine is offsite and you have them both connected via [Tailscale](https://tailscale.com), you can use the Tailscale IP here, *even if the machine is local*. Tailscale connections are designed to be peer-to-peer, so it should stay in the same local network and get the full wire rate.
6. For username, change it to the username of the user you created on the REMOTE system. That is, the PRODUCTION system.
7. Select the private key you created in step 2.
8. Click "Discover remote host key".
9. Click Save.

### On the production system:

1. Navigate to Credentials > Users. Edit the user you created for your backup tasks.
2. Take the public key you copied earlier and paste it into the Authorized Keys field. (Or upload it if you saved it as a file.)
3. Click save.

## Giving users needed permissions

1. Because we're not using the root user, we need to give our backup users the ability to run certain commands with sudo, without a password. **None of these commands can be used destructively.** 
2. Navigate back to Credentials > Users. Edit the backup user.

### On the backup system:

3. Type the following into the "Allowed sudo commands with no password" box. The asterisks are wildcards.

```
/sbin/zfs mount *
/sbin/zfs create *
/sbin/zfs receive *
```

### On the production system:

3. Type the following into the "Allowed sudo commands with no password" box. The asterisks are wildcards.

```
/sbin/zfs send *
/sbin/zfs snapshot *
/sbin/zfs list *
/sbin/zfs get *
```
This will allow a completely unprivileged user to run the backups for us.


## Creating the replication task

### On the backup system:

1. Navigate to Data Protection > Replication Tasks. Click Add.
2. For the source location, choose "On a different system".
3. Choose the SSH connection we created earlier.
    - When prompted if you would like to use sudo for ZFS commands, allow it. This is needed because the user we're doing this operation with does not have admin privileges.
4. For the source, choose the dataset that you would like to back up. I usually also check off the recursive box, though this has no effect unless you've created nested datasets.
5. For the destination, choose either the pool itself or the dataset ABOVE the one you want to replicate to. **The dataset CANNOT exist prior to the first replication. It will fail if you create it in advance.** You'll need to type in the name of the dataset in the text box. This is slightly unintuitive, I know. I don't know why they did it this way.
6. Choose a schedule and a retention policy.
7. Run your first replication. The first one will take a long time if there is a lot of data to copy, but subsequent runs will take only as much time as it takes to send the changed blocks down the wire.

Keep in mind when choosing your retention policies on the source and destination systems that **the source and destination systems NEED to have at least one snapshot in common!**

Happy replicating!
