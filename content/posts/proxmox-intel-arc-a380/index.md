+++
title = 'Passing through Intel Arc A380 to a Proxmox VM'
date = 2025-08-17T18:02:07-04:00
draft = false
+++

I recently purchased an AsRock Challenger ITX Intel Arc A380 GPU to use for my video transcoding needs.

It was a bit of a headache to get it passed through to a VM in Proxmox, so here's a log of what worked for me, in hopes that it will help someone else down the road.

My VM is Ubuntu 24.04 LTS, running kernel 6.14 (HWE kernel).

Proxmox is version 8.4.

I was able to successfully pass through the GPU to the VM, but it shows as `UNCLAIMED` in `lshw`:

```bash
root@gputest:/home/scott# lshw -c display
*-display UNCLAIMED
description: VGA compatible controller
product: DG2 [Arc A380] vendor: Intel Corporation
physical id: 10 bus info: pci@0000:06:10.0
version: 05 width: 32 bits
clock: 33MHz
capabilities: msi pm vga_controller cap_list
configuration: latency=0
```

I'm not going to go through all the troubleshooting I did, just share what worked.

The A380 is very new, so we need to force the kernel to load the XE driver.

In addition to this, because I'm using [cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) and one of Ubuntu's [Cloud Images](https://cloud-images.ubuntu.com/), I can't simply specify the kernel parameter in `/etc/default/grub`. If you're not using cloud-init, ignore the next step and simply specify this in `/etc/default/grub` like so:

`GRUB_CMDLINE_LINUX_DEFAULT="quiet i915.force_probe=56a5`

Then run `update-grub` and reboot.

{{% callout note %}}
These steps may be different if you're using a distro like Fedora - I believe they now use `grubby` to accomplish this task.
{{% /callout %}}

However, for those of us using cloud-init, you can instead simply specify these parameters in `/etc/modprobe.d/`

`sudo vim /etc/modprobe.d/i915.conf`

{{% callout note %}}
2025/08/23: This originally was set to load in the Xe driver and disable i915, but I found out later that things like Jellyfin do not have support for Xe, at least not yet. For now, you must force_probe the i915 driver instead.

As of today I have tested this with both Jellyfin and Plex and can confirm the card works as expected.
{{% /callout %}}

```
options i915 force_probe=56a5
```

Now that that's done, here are my PCI passthrough settings on the VM:

```shell
root@pve01:~# qm config 100
<snip>
cpu: host
hostpci0: 0000:2f:00,pcie=1 <---- This is the GPU itself
hostpci1: 0000:30:00,pcie=1 <---- This is the GPU's audio controller.
<snip>
```

It does _not_ need to be set as the primary GPU, but you do want your machine type to be q35 and you want to enable PCI express.

You also want to give the VM _more_ than 4GB RAM (so it can initialize the large BAR).

And now on to the thing that took me hours to figure out.

<span style=color:red>CSM _needs_ to be disabled in BIOS</span>. At least with my motherboard, leaving it on meant that resizable BAR was disabled. Without this, passthrough didn't work at all.

Other settings you should enable:

- Above 4G Decoding (May also be called large memory allocation).
- SR-IOV
- IOMMU

Most of these can be found either in your PCI subsystem or chipset settings.

Now the card shows normally in `lshw`.

```bash
scott@gputest:~$ sudo lshw -c display
  *-display
       description: VGA compatible controller
       product: DG2 [Arc A380]
       vendor: Intel Corporation
       physical id: 0
       bus info: pci@0000:01:00.0
       version: 05
       width: 64 bits
       clock: 33MHz
       capabilities: pciexpress msi pm vga_controller bus_master cap_list rom
       configuration: driver=xe latency=0
       resources: iomemory:38000-37fff irq:38 memory:c0000000-c0ffffff memory:380000000000-3801ffffffff
```

I'm not sure that this is 100% needed, but Intel also recommend [installing their repo and some of their support packages](https://dgpu-docs.intel.com/driver/client/overview.html#ubuntu-latest). It can't hurt, so why not.

```bash
sudo apt-get update
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:kobuk-team/intel-graphics
sudo apt-get install -y libze-intel-gpu1 libze1 intel-metrics-discovery intel-opencl-icd clinfo intel-gsc
sudo apt-get install -y intel-media-va-driver-non-free libmfx-gen1 libvpl2 libvpl-tools libva-glx2 va-driver-all vainfo
```

For some reason the `intel_gpu_top` and `clinfo` tools don't seem to recognize the card. Possible that they just haven't been updated with support for the newer Xe driver over i915.

However, we can confirm that things are working with `vainfo` and `ffmpeg`.

I actually took this a step further and built a Docker image with `vainfo` and `ffmpeg` (since again, my intention is to use this for [Jellyfin](https://jellyfin.org/) in Docker.)

```bash
root@f7fe7efcfa51:/# vainfo
Trying display: wayland
error: XDG_RUNTIME_DIR is invalid or not set in the environment.
Trying display: x11
error: can't connect to X server!
Trying display: drm
libva info: VA-API version 1.22.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_22
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.22 (libva 2.22.0)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 24.3.4 ()
vainfo: Supported profile and entrypoints
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileNone                   : VAEntrypointStats
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSliceLP
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSliceLP
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointEncPicture
```

```bash
root@f7fe7efcfa51:/# ffmpeg -hide_banner -encoders | grep qsv
 V..... av1_qsv              AV1 (Intel Quick Sync Video acceleration) (codec av1)
 V..... h264_qsv             H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 (Intel Quick Sync Video acceleration) (codec h264)
 V..... hevc_qsv             HEVC (Intel Quick Sync Video acceleration) (codec hevc)
 V..... mjpeg_qsv            MJPEG (Intel Quick Sync Video acceleration) (codec mjpeg)
 V..... mpeg2_qsv            MPEG-2 video (Intel Quick Sync Video acceleration) (codec mpeg2video)
 V..... vp9_qsv              VP9 video (Intel Quick Sync Video acceleration) (codec vp9)
```

Me IRL this weekend.

![IT Crowd Moss Throwing Computer](images/throw-computer.gif)
