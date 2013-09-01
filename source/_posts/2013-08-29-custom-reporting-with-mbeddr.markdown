---
layout: post
title: "custom reporting with mbeddr"
date: 2013-08-29 22:21
comments: true
published: false
author: Kolja Dummann
categories: 
- mbeddr
- tutorial
---

As some of you might already know I have worked on an Arduino extension for mbeddr. Because such system run headless you can't really use `printf`. These system usually ship with a serial or USB port which can be used to communicate with the world. There are actually two ways to deal with this, first we could replace the printf backing filehandle with a handle that writes to the serial port, or second use mbeddrs own reporting infrastructure to deal with this. The mebddr approach also give us the more flexibility because we could either write messages to the seiral port or store critical errors in the EPROM for further investigation.