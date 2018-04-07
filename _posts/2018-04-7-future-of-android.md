---
layout: post
comments: true
title: Thoughts on Future of Android
tags: [Android]
---

Last week XDA reported that uncertified devices will not be able to run Google apps anymore [1]. As highlighted, custom ROMs were left out of these newly imposed restrictions but (all devices) had a new 100 device registration limit*. Such restrictions are warranted as smart phones continue to integrate deeper into people's life and need to be more secure than ever before because of piracy concerns by copyright holders and security concerns of payment apps. 

That said, in my opinion, Android is moving to a closed ecosystem with Google trying to emulate Apple's walled garden. In this blog post I try to explain why I think so and my opinion about it as an Android developer.

In 2015, Google released a new version of Android called Marshmallow and along with it came one of the biggest changes of the OS - Doze mode[2]. Doze mode endeavoured to fix Android's battery problem. With completely free reign over background processes and the license to wake up devices whenever they fancy to, developers abused Android's rather liberal resource allocation policy leading to Android devices gaining a bad reputation for having a pathetic battery life. 

Doze was a welcome change. The OS would put limitations on an app's background activity and it's ability to acquire wake-locks. But Google gave a "master key" to developers in the form of FCM/ GCM priority notifications. Any app can momentarily escape doze mode if it receives a high-priority FCM message [3]. Thus, Google successfully forced a private API onto an open source operating system. 

Commonsware explained the situation eloquently -

> Beyond this, intentionally altering an open source operating system to steer people towards proprietary APIs reeks of Mafia-style behavior (“Gee, that’s a nice app you have there. Pity if something were to happen to it”). If push messaging on Android were an open, pluggable system, with multiple competing implementations, I would have no problem with Android giving benefits towards apps that use push. But third-party push engines (e.g., XMPP, WebSockets-based long polling) are going to be subject to “Doze mode” and “app standby” and cannot help apps get the data they need when the user asked for that data. Here, GCM’s exalted status, and Android’s dependence upon it, represents damage to be routed around.

Over the subsequent versions of Android, Google made doze mode stricter and more aggressive while FCM remains the only "master key".

In the next two years, Google introduced SafetyNet [4] and Google Play Protect [5]. The former is aimed to enable developers detect whether a user's phone has been tampered with or not (read root) and the latter enables Google to scan a user's smart phone and the play store for potentially malicious apps. 

SafetyNet left a lot of rooted users unable to download apps like Netflix etc. via the playstore until rescue came in the form of Magisk [6]. Play Protect seems to be quite intrusive - it constantly scans all the apps present on a device and sends that information to Google; potentially paving a way for Google to block even side-loaded apps on Android (this statement is purely speculative). Currently, Play protect can be disabled via system settings. 

In March, this year, Google released the first developer preview of the next Android version - Android P. Immediately it was discovered that Google has dropped support for overlays [7]. Theming and customising is a major part of the Android ecosystem and with Oreo, Google officially introduced support for overlays (previously, one had to root to make it work). Then, took it away in the preview of the next version in such a way that even a rooted device won't be able to implement it. Of course P has not been locked down yet and it may very well come back but the likeliness of that happening seems really low.

Google seems to be moving to a more controlled and closed environment which is understandable given that people (and even Google) want a more secure, stable ecosystem which respects their privacy. Personally, as long as the custom ROM & root community is not actively discouraged, given enough breathing space and not encourage cat and mouse games, I think Android will do fine for casual and enthusiast users.

On the other hand, for me, Android ecosystem is moving to a less diverse in term of device specifications. Later this year I'll probably be in the market for a new phone. I prefer smaller/ compat phones (<5 inches screen) with unlockable bootloaders that can be rooted and support custom ROMs. Unfortunately, I can't seem to find a device to match my needs anywhere. Nokia 5 comes really close but Nokia seems to have no interest in unlocked bootloaders. I'm running out of devices which are not as big as a tablet.  


*Google later removed this limit. [8]


[1] https://www.xda-developers.com/google-blocks-gapps-uncertified-devices-custom-rom-whitelist/ 

[2] https://commonsware.com/blog/2015/06/03/random-musing-m-developer-preview-ugly-part-one.html

[3] https://developer.android.com/training/monitoring-device-state/doze-standby.html#using_fcm

[4] https://android-developers.googleblog.com/2017/09/safetynet-verify-apps-api-google-play.html

[5] https://www.blog.google/products/android/google-play-protect/

[6] https://forum.xda-developers.com/apps/magisk

[7] https://www.androidauthority.com/android-p-blocks-substratum-themes-custom-overlays-844447/

[8] https://www.xda-developers.com/google-removes-100-device-registration-limit-uncertified-device-page/