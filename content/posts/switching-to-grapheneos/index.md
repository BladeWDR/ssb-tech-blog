+++
title = 'I switched to GrapheneOS'
date = 2026-03-28T15:41:08-04:00
draft = false
+++

# I switched to GrapheneOS

I recently purchased a 'new-to-me' Pixel 10 Pro to replace my Pixel 8 Pro.

There was nothing particularly wrong with the 8, but I'd had it for a few years, and my father is in need of a new phone, so it was a good opportunity for me to upgrade.

At the same time, I've also been meaning to check out [GrapheneOS](https://grapheneos.org/), a custom ROM based on AOSP that has a focus on privacy and security.

Given Google's recent track record and their initiatives to [lock down Android under the pretext of security theater](https://developer.android.com/developer-verification) I figured there had never been a better time to try out GrapheneOS.

This post is a record of my experience and of my first impressions.

## Installation

I used the official [WebUSB installer](https://grapheneos.org/install/web) method to install GrapheneOS.

It was extremely painless, especially compared with my experience installing CyanogenMod in the old days.

All I needed to do was install one Android developer package on Fedora, open up Chromium and follow the prompts.

I was up and running within 15 minutes, which is very impressive.

## First Impressions

Landing on the home screen at first launch is somewhat intimidating.

The default GrapheneOS home screen is _very_ utilitarian and barebones.

However, I was able to install [Obtanium](https://obtainium.imranr.dev/) without much trouble and get most of the apps I wanted installed.

However, I do have some need for Google Play and Play Services, so I installed those too. GrapheneOS makes it painless to do so by including a mirror of the official APK files in their own App Store. I know this compromises the privacy of the device somewhat, but as always one must take into account their own risk profile and ride the line between convenience and security.

I do enjoy that GrapheneOS gives the user far more control over what privileges every app on the device has, even for Google Play Services.

So far I haven't had too many issues.

The only problems I've run into have been the Reddit client that I prefer to use, and of course Google Wallet.

I had occasionally used Google Wallet to do NFC payments, but I knew that this was likely to not work on a custom ROM.

The reddit client I was using was [Relay for reddit](https://play.google.com/store/apps/details?id=reddit.news&hl=en_US). For some reason the Google Play Store showed that the app was not compatible. I found a way to get the application to install via the [Aurora Store](https://store.auroraoss.com/) and the app launched just fine - however I was unable to log in at all.

The login webview came up as normal, but no matter what I did, it claimed that my password was incorrect, despite the same one perfectly fine in both the website and Reddit's own official app.

I can only assume this has something to do with the Play integrity API.

Frankly I don't use reddit much anymore, so I simply uninstalled both apps and will use the mobile website going forward.

## Final Thoughts

I may come back and update this post later, or publish a follow up.

So far I am very impressed with what I've seen.

As far Android as a whole - I very much wish that there were more open-source alternatives.

I'm aware of projects like [PostmarketOS](https://postmarketos.org/) and [SailfishOS](https://sailfishos.org/), but the issue is two things - device compatibility, and application support.

Everyone should support the [Keep Android Open](https://keepandroidopen.org/) initiative!
