---
layout: post
comments: true
title: Tracking time spent viewing RecyclerView items
tags: [Code, RecyclerView, Android]
---

There are multiple analytics tools for providing insightful information with respect to user behaviour, bugs and crashes on an app. One parameter that seemed to be missing in most free analytic tools is the time spent looking at a certain view in Android and then providing the raw data of it.

In this post I explain how to track the time a user spends looking at certain items in a RecyclerView.

We assume that any item, within the RecyclerView, with a certain minimum visibility, for a certain time on the screen is being viewed by the user. For example, suppose an item with 100% of its height and width within the screen of the device for greater than 3 seconds is being shown. We assume that that such an item has captured the users attention and hence, should be tracked.

We'll need a RecyclerView and add `OnGlobalLayoutListener` and `OnScrollListener` to it to make our tracking work.

`OnGlobalLayoutListener` is needed to detect when the RecyclerView has successfully been populated by the adapter for the first timer and when the developer wants the tracking to stop.

`OnScrollListener` is required for tracking all the items the user might see on scrolling down and back up.
