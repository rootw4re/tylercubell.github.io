---
title: 'Running Out of Memory With Too Many Apache VirtualHosts'
author: Tyler
layout: post
permalink: /running-out-of-memory-with-too-many-apache-virtualhosts
categories:
  - Posts
---

I'm running a VPS with 1GB of RAM that hosts around 20 low-traffic sites (LAMP stack) and I was having a problem where requests started taking longer and longer and MySQL would sporadically crash. I upgraded PHP and Apache, tweaked some MySQL settings, and installed `monit` to keep the MySQL process alive but it turned out these changes were treating the symptoms and not the cause. It was inexplicable because in the beginning every website was loading quickly and I hadn't made any significant changes since the VPS was created.

Later, taking a look at the memory usage, I discovered it was always at or near 100% and using the swap. When the VPS got a burst of traffic I guess the swap was eaten up too and the MySQL process got killed. So that explains one piece of the puzzle but I still didn't understand what was taking up so much memory in the first place. One of the things I tried was looking at the Apache access and error logs to see if there were any clues there. I didn't see anything out of the ordinary and I cleared everything out to start fresh. Instantly memory usage dropped to 50% and the sites were loading instantly again. WTF?

Turns out `other_vhosts_access.log` was the bottleneck and clearing that solved the problem. Going forward I'll either be rotating logs or disabling that particular log.
