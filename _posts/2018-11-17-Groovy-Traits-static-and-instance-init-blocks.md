---
layout: post
title: Groovy Traits static and instance init blocks
tags: java groovy traits
category: java
---

# Groovy Traits static and instance init blocks

Welcome to Intinite Technology ∞ Blog and this is our first post!

Today we will show a very small example of a new useful Groovy Traits feature: 
- Multiple inheritance of static and instance init blocks

Note: it seems that this is first post about this feature as it was implemented just a few days back thanks to Groovy Team and [paulk_asert](https://groovy-community.slack.com/team/U2P6GPHHC).

Let's consider below use case:
* Several classes extend java.lang.Thread and are instantiated into a limited number of instances
* Every instantiation increments a static counter ("instance #")
* Upon instantiation, it is needed to automatically set thread name to Simple Class Name + instance #, e.g.:
    - SenderThread1, SenderThread2, etc...

