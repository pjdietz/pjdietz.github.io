---
layout: post
title: Android Delayed Execution on UI Thread
---

Ever need to run something on the UI thread after a short delay? Here’s an example. Create a new thread that will block for a set amount of time, then create a Runnable and send it to the Activity’s `runOnUiThread()` method.

Adapted from [this Stack Overflow answer](http://stackoverflow.com/questions/3247554/how-to-show-a-view-for-3-seconds-and-then-hide-it).

{% gist 5488864 %}
