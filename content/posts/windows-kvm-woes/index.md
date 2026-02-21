+++
title = 'Issues transferring Windows 11 VM from KVM to Proxmox'
date = 2026-02-20T21:11:38-05:00
draft = false
+++

I ran into a strange issue when trying to transfer a Windows 11 Pro virtual machine from KVM on my Fedora desktop to Proxmox.

The VM was using a VirtIO block device for it's storage as I normally do.

I transferred the VM qcow2 file to Proxmox and created a new VM with matching hardware, the imported the disk with:

```bash
qm importdisk 300 win11.qcow2 fast
```

(`fast` being the name of my SSD zpool.)

However when I went to boot the VM, it refused to boot at all.

Originally, it seemed to be some sort of issue with the EFI bootloader.

I was able to boot into a Windows install disk and recreate the EFI partition with `diskpart` and `bcdboot`.

Once I'd done that, the VM attempted to boot Windows. Rejoice... or not.

Nope, now it's getting an `INACCESSIBLE BOOT DEVICE` BSOD.

I tried innumerable things without success, the golden ticket wound up being this [superuser.com thread](https://superuser.com/questions/1057959/windows-10-in-kvm-change-boot-disk-to-virtio).

I had to first:

- Change my boot disk to use SATA instead of VirtIO.
- Boot into Windows.
- Enable Safe Mode `bcdedit /set "{current}" safeboot minimal`.
- Shut down the VM, change the boot disk back to VirtIO.
- Let it boot up into Safe Mode.
- Revert to normal boot. `bcdedit /deletevalue "{current}" safeboot`

I still don't quite understand what the problem was here.

Windows has the concept of "boot start" drivers, yes, where it only loads a subset of the available drivers at first boot, but I had been using the VirtIO drivers since this OS was first installed, so why was this suddenly a problem?

Apparently, rebooting into Safe Mode this way forces Windows to load all available boot start drivers, and because VirtIO worked here, it then remembered that for the next, normal boot.

If anyone has an actual explanation for why this happened I'd appreciate it.

I wasted _far_ too much time on this, I could have wiped and reloaded the VM many times over in the time it took me to fix this, but it bothered me far too much to let it go.

This kind of nonsense makes me glad I've switched to using almost exclusively Linux in my personal life.
