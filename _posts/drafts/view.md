---
layout: post
comments: true
title: Tracking time spent viewing RecyclerView items
tags: [Code, RecyclerView, Android]
---

There are multiple analytics tools for providing insightful information with respect to user behaviour, bugs and crashes on an app. A parameter that seemed to be missing in most free analytics is the time spent looking at a certain view in Android and then providing the raw data of it.

In this post I explain how to track the time a user spends looking at certain items in a RecyclerView.

We assume that any item, within the RecyclerView, with a certain minimum visibility, for a certain time on the screen is being viewed by the user. For example, suppose an item with 100% of its height and width within the screen of the device for greater than 5 seconds is being shown. We assume that that such an item has captured the users attention and hence, should be tracked.
