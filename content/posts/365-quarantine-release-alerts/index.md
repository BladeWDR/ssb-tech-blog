---
title = 'PSA - alert policies in Office 365 depend on audit logging'
date = 2024-12-27T13:18:28-05:00
draft = false
---

This post is a bit of a PSA of sorts.

I spent at least an hour trying to figure out why this wasn't working.

As it turns out, in order for this feature to work, you'll need to first [turn on audit logging in your Office 365 tenant](https://learn.microsoft.com/en-us/purview/audit-log-enable-disable?tabs=microsoft-purview-portal#turn-on-auditing).

Maybe this seems obvious to some - it wasn't at all obvious to me. I wrongly assumed that these events would be raised in some way - not that the alerts were just tracking the user activity logs.

The Microsoft documentation I referenced above also claims that logging is enabled by default - but on every 365 tenant I checked (and through my work I have access to more than 2 dozen) it was NOT turned on by default, even on several newly created tenants. This feature really should be on by default.

Once you enable the feature it may still be up to 24 hours before your alerts start working.

I'm currently using this feature to track when users submit requests to release messages from Quarantine - anyone else have some useful policies? Share down in the comments.
