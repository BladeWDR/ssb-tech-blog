+++
title = 'PSA: Unifi IPSec site-to-site tunnels and DHCP'
date = 2025-04-01T13:09:28-04:00
draft = false
+++

Bit of a PSA of sorts, as this is the second time that I have run into this particular issue.

Scenario: You had a Unifi gateway such as a UDM Pro with a static IP address. You switched your WAN IP from static to dynamic, for whatever reason. (In my case it was a different ISP that doesn't hand out actual static IPs, only DHCP reservations.)

You may find that the router holds onto that old IP, and if you're like me, you'll be confused as hell.

The cause is your IPSec site-to-site tunnel.

You'll need to make note of any settings it had (Pre-shared keys, remote endpoints, remote subnets, etc), delete the tunnel completely, and then once you've successfully gotten your new IP from DHCP, recreate the tunnel from scratch.

Once you remove the tunnel, you should get an IP from DHCP almost immediately.
