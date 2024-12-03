+++
date = '2024-12-02T23:57:12-05:00'
draft = false
title = "Adventures in DNS (But it's actually DHCP this time.)"
+++

So, story time.

Boss has a friend who does IT for an electrical contractor 30 mins or so away from our primary office, and he brings us in because he’s having DNS issues he can’t figure out

Get onsite there and go over his setup – typical mess of a network closet with no brand consistency and mismatched patch cables, whatever. Otherwise looks good from a network perspective.

However, he mentions that he has no access to the firewall since it’s owned by the ISP.

Personally, I would have been on top of the ISP until they either gave me the ability to log into that thing, or had them put it in bridge mode and installed my own firewall. I do not like and do not trust any firewall that I do not control. Sorry, it's just the paranoid sysadmin in me.

So we finally start looking at the problem and the issue is that he keeps having to hardcode his DNS settings and he’s having weird WiFi connectivity issues. Everything points to DHCP not setting the DNS settings consistently… which made me suspect a rogue DHCP server.

So I connect my laptop and run an ifconfig – and I see that DHCP is coming from some IP address 10.0.0.69. Knowing at this point he didn’t have access to the firewall, I asked him if that was one of his servers, he says no. I asked him where he WAS running DHCP, and was told it was on one of the domain controllers, but he didn’t remember which one. Red flag #1.

I do an nmap scan to see if I can grab the vendor of the device that has that IP – it reports back that it’s from Axis Communications. Not being familiar with it, I asked about that vendor, and was told that that was the brand of their security cameras and NVR.

I go okay, so there’s your problem right there. You have 2 DHCP servers, and it’s a question of which one responds first to the DHCPDISCOVER broadcast.

So we go look at the cameras – no passwords for ANYTHING. we lucked into finding the password for the computer controlling them written on a piece of paper near the rack it was installed in.

Eventually I figured out that one of the cameras had DHCP running, and we turned it off, but then we tried getting a new DHCP lease – it got one, but in the wrong subnet entirely.

This of course, understandably confuses me. I noticed that the DHCP server is now 192.168.0.1, which isn’t even the right subnet. What the deuce?

Ran another network scan, that IP has the same vendor as before, but a different MAC address now, so it’s a totally different device.

Turns out, the appliance they have running their cameras? it’s 2 devices in one – a windows PC acting as the controller, and the built in switch which has its own DHCP server (and presumably it’s own operating system and mainboard).

Of course, no password for that either, so I Googled and found it’s on a sticker on the bottom of the unit. I found the password, logged in and turned off DHCP. That’s it, right? We’ve fixed it?

Tried to get a lease once more… nothing happens at all.

Turns out… he was NOT running DHCP on the domain controller. Or anywhere except for this NVR, that had it turned on by default.

I installed the DHCP role on one of the domain controllers and configured it with the proper settings, and a reasonable scope, along with primary and secondary DNS for their domain. Did another ipconfig /renew on one of their machines and… success! We got an IP in the correct subnet, and it is assigning the correct DNS servers to the client machines. WiFi works perfectly as well!

How the network got into this state and stayed operational for as long as it did? Who can say? Miracles happen every day, right?

Like they say, it’s always DNS. But sometimes it’s not DNS, sometimes it’s badly misconfigured DHCP.
