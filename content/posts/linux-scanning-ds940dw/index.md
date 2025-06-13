+++
title = 'Setting up brscan-skey with the Brother DS-940DW on Ubuntu'
date = 2025-06-13T18:58:09-04:00
draft = false
+++

# Setting up brscan-skey with the Brother DS-940DW on Ubuntu

## Overview

The Brother DS-940DW is a little handheld mobile scanner. Getting it working on Linux in a way where you can actually use the Start/Stop button on the scanner to scan things was a bit of a bear, so I thought I'd document how I did it.

Part of the issue is that brscan-skey depends on the old version of `libsane`, and does not work with `libsane1`.

## Solving the dependency problem

First, install the provided driver from the [support page for the DS-940DW](https://support.brother.com/g/b/downloadlist.aspx?c=us&lang=en&prod=ds940dw_us_eu_as&os=128)
You can grab the scan-key-tool deb package from [here](https://download.brother.com/welcome/dlf006652/brscan-skey-0.3.4-0.amd64.deb).

Going by [this Brother support article](https://support.brother.com/g/b/faqend.aspx?c=as_ot&lang=en&prod=dcpl2600d_eu_as&faqid=faq00100811_000), I was able to get brscan-skey installed.

### Steps

```
sudo su
apt update
apt install sane libsane1 sane-utils imagemagick
wget -c http://jp.archive.ubuntu.com/ubuntu/pool/universe/s/sane-backends/libsane_1.1.1-5_amd64.deb
dpkg -i libsane_1.1.1-5_amd64.deb
apt install evince
```

## Configure scanning

I had all sorts of fun figuring out how this all works, and honestly it's pretty simple.

Pressing the Start/Stop button on the scanner while brscan-skey is running calls this shell script:

`/opt/brother/scanner/brscan-skey/script/scantofile.sh`

You can have it do whatever you want by editing the script.

By default it uses psutils and pod2pdf, and a bunch of other utilities to scan first to PostScript, then convert it to PDF.

I don't often have the need to scan complex documents, so I simplified this quite a bit, as I'll show you here in second.

But before we do anything, we need to add some udev rules.

Create a file at `/etc/udev/rules.d/NN-brother-mfp-brscan5-1.0.2-2.rules` with the following content:

### udev rules

```
ACTION!="add", GOTO="brother_mfp_end"
SUBSYSTEM=="usb", GOTO="brother_mfp_udev_1"
SUBSYSTEM!="usb_device", GOTO="brother_mfp_end"
LABEL="brother_mfp_udev_1"

ATTRS{idVendor}=="04f9", GOTO="brother_mfp_udev_2"
GOTO="brother_mfp_end"
LABEL="brother_mfp_udev_2"

ATTRS{bInterfaceClass}!="0ff", GOTO="brother_mfp_end"
ATTRS{bInterfaceSubClass}!="0ff", GOTO="brother_mfp_end"
ATTRS{bInterfaceProtocol}!="0ff", GOTO="brother_mfp_end"

MODE="0666"
GROUP="scanner"
ENV{libsane_matched}="yes"
LABEL="brother_mfp_end"
```

Reload udev rules:

`sudo udevadm control --reload && sudo udevadm trigger`

### Scanner group

You'll also want to add the user you want doing the scanning to the scanner group.

`sudo usermod -aG scanner <username>`

### Systemd service

I created a systemd service to run brscan-skey at boot.

Create the following at `/etc/systemd/system/brscan-skey.service`

```
[Unit]
Description=Brother scan-key-tool

[Service]
User=scott
Type=forking
ExecStart=/opt/brother/scanner/brscan-skey/brscan-skey
ExecStop=/opt/brother/scanner/brscan-skey/brscan-skey --terminate

[Install]
WantedBy=multi-user.target
```

Run the following commands:

```
sudo systemctl daemon-reload
sudo systemctl enable --now brscan-skey.service
```

You may need to reboot. I did.

### Simple scantofile.sh script

First make a backup of the existing script, just in case.

`sudo mv /opt/brother/scanner/brscan-skey/script/scantofile.sh /opt/brother/scanner/brscan-skey/script/scantofile.sh.bak`

Now we create our own version:

`sudo vim /opt/brother/scanner/brscan-skey/script/scantofile.sh`

```sh
#! /bin/sh
set +o noclobber

BASE=$HOME/brscan

pdf_name=$(date | sed s/' '/'_'/g | sed s/'\:'/'_'/g)
output_full="$BASE/$pdf_name"

# scan the image in using scanimage
scanimage -x 215.9 -y 279.4 --format=png > "$output_full".png

# Convert to PDF using imagemagick
convert "$output_full".png "$output_full".pdf

echo "Created file $output_full.pdf"

# Cleanup

rm "$output_full".png
```

My script assumes a lot of things. It assumes that you're using letter size paper, for one. This was a really quick and dirty hack - maybe I'll refactor this in the future.

## Future goals

My goal for getting this working was to have a small, low power x86 PC that just runs this and nothing else. It's connected via WiFi so I can put it anywhere, and hopefully make it so that I _actually use it_.

I have it exposing an NFS share that I'm going to mount as an ingestion folder for [Paperless-NGX](https://docs.paperless-ngx.com/).

Maybe I'll create another blog post soon showing how I got that set up.
