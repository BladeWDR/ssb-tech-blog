+++
title = 'Linux on an Asus CR11 Chromebook'
date = 2025-12-13T15:52:08-05:00
draft = false
+++

## In which I have zero self control

While browsing the Amazon Black Friday sales a few weeks ago, a particular deal caught my eye.

An ASUS Chromebook CR11 ruggedized laptop for around $130.

Now, I have enough laptops, but I've always wanted to try and hack Linux onto one, and this one was attractive due to its ruggedized nature as well as a full blown Intel N100 CPU.

I purchased the laptop and once it arrived, I began poking around the internet to see how I could get regular Linux on this thing.

## Installing Linux, Part 1

I have no desire to use Google's spyware filled ChromeOS, so if I couldn't find a way to get Linux on this machine, it was going back to Amazon.

A few searches guided me to [mrchromebox.tech](https://docs.mrchromebox.tech/).

This specific model I had was very similar to a few other builds that show full compatibility, so my hopes were raised.

I went through their [Getting Started](https://docs.mrchromebox.tech/docs/getting-started.html) guide, which helped me get Developer Mode enabled, and I found my device ID.

`ANRAGGAR`

This HWID showed full compatibility with the full UEFI replacement firmware, which is exactly what I wanted.

However, on most ChromeOS devices, it's a requirement to [disable hardware write protection](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html) in order to flash the replacement firmware.

This particular model had a Ti50 security chip, so the only method of disabling write protection was to use Closed Case Debugging (CCD). This process requires a "SuzyQ" ChromeOS debug board.

A little bit annoying, but not the end of the world.

I went on eBay and found the debug board listed [here.](https://www.ebay.com/itm/316024978790).

It took around a week to get to my house, and with that in hand, I tried again.

## Installing Linux, Part 2

I followed the steps listed [here](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#using-closed-case-debugging-ccd-using-a-suzyqable) to disable hardware write protection.

{{% callout note %}}
**NOTE**
- A SuzyQ cable is not like a typical USB-C connected device, where it is reversible. If you look closely at the board, one will be labeled side A, the other is side B. On the board that I received, side A needed to be facing up.
- I had plugged it into the top left USB-C port on my laptop, I'm not sure if it will work when plugged into the other one. However, I did power the board by plugging it into the other available USB-C port.
- The MrChromebox documentation says that it takes 2-3 minutes of pressing the physical presence sensor to enable CCD. In reality, it was more like 5. It paused several times during the process, and it rebooted without ever showing me the "PP Done!" message that the docs suggest. However, the steps there did work fine.
{{% /callout %}}

Once I had gone through the steps and verified that hardware write protection was indeed disabled, I proceeded to the next step, which was flashing the firmware.

I used the provided [firmware utility script](https://docs.mrchromebox.tech/docs/fwscript.html) to flash the firmware.

Generally I don't recommend running scripts blindly from the internet, but it truly does make this process a ton easier, and if you wish, you can review the script yourself. (And I recommend doing so before you run it!)

First, I backed up the existing ChromeOS firmware to a USB stick. Handy, in case I ended up needing to return this laptop.

Then, I proceeded to flash the Coreboot UEFI firmware.

It took a few minutes, but once done I was able to boot into the [Fedora 43 Sway Spin](https://fedoraproject.org/spins/sway/download) installer.

## Some problems

I noticed right away that the touchpad didn't work in the installer, but I proceeded anyway. I could have grabbed a mouse, but being the lazy person I am, I just navigated the installation with the keyboard instead.

Once installed, it took me a few minutes to figure out where all the needed buttons were. 

Linux maps the Chromebook Search key to Super_L, which I needed to open the default `drun` menu. Can't do much in [Sway](https://swaywm.org/) without being able to open `drun` or a similar launcher!

The touchpad still didn't work, and neither did sound. However, everything else worked a treat.

It was a bit strange, since the touchpad appeared normally in `libinput --list-devices`, and the sound driver appeared normally in `pavucontrol`, however, neither of them worked at all, and there were a TON of errors in `dmesg` regarding the sound driver.

I did some searching on the [chultrabook forums](https://forum.chrultrabook.com/), and I was able to find solutions for both issues.

### Fixing the audio problem

The [chultrabook docs](https://docs.chultrabook.com) contained a "Post-Install" FAQ that was very helpful - I found their [audio script](https://github.com/WeirdTreeThing/chromebook-linux-audio) which was all I needed to get sound working.

### Fixing the touchpad problem

The touchpad issue was a bit trickier to pin down.

Eventually I found [this post](https://forum.chrultrabook.com/t/no-touchpad-multitouch-on-asus-chromebook-cx9-drobit/3606) that pointed me in the right direction.

The answer was to configure a `libinput` quirk:

```bash
sudo vim /etc/libinput/local-overrides.quirks
```

Enter the following content:

```
[PNP Touchpad]
MatchName=*PNP*Touchpad*
AttrResolutionHint=31x31
AttrPressureRange=10:8
```

Replace `PNP` with your touchpad model as it shows up in `libinput --list-devices`.

Once you've done so, write and quit the file, then reboot your Chromebook.

Success! The touchpad now works!

## Final Thoughts

The Chromebook performs very well under Fedora, but I'm also not running a full on desktop environment.

If KDE is your preferred environment, you may run into more issues, especially since this device only has 4GB of onboard memory, and a relatively weak Intel CPU.

I haven't used Gnome in quite some time, but I expect it would run into similar limitations.

The screen isn't fantastic, but it's good enough for what I'm planning on using this laptop for - effectively just tossing it in my bag when I need a super small and light laptop for travelling, or when I want to sit on the couch and chat on Discord while watching TV.

_Do I recommend buying this specifically to run Linux?_

No, not really.

It works great once you get it going, especially for the price, but you can likely find better laptops on eBay.

Used ThinkPads are the way to go for cheap laptops.

This was just a fun little "I wonder if I can do this" project.

That said, if you already have one of these devices? I _highly_ recommend installing regular Linux on it.
