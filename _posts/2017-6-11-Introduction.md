---
layout: post
title: Introduction
---

# Welcome to my blog!
Hello everyone,

This is my blog where I will write about the development progress of my GSoC 2017 [project](https://summerofcode.withgoogle.com/projects/#4983264524632064)
with the Intel Media and Audio for Linux organization.

I want to thank my mentors Víctor Jáquez and Sreerenj Balachandran for the help and suggestions they have provided.

# Project goals
My project will mainly involve work in
the h264 decoder of the [gstreamer-vaapi](https://cgit.freedesktop.org/gstreamer/gstreamer-vaapi/) project
and the h264parse plugin of [gst-plugins-bad](https://gstreamer.freedesktop.org/modules/gst-plugins-bad.html).

According to the H.264/AVC standard, a decoder should be able to only decode the base view of an MVC encoded stream if the hardware does not support MVC decoding.
The same applies to the base layer of a SVC encoded stream.
I propose adding proper backwards compatible decoding support to the vaapih264dec plugin by resolving bugs
[#732267](https://bugzilla.gnome.org/show_bug.cgi?id=732267),
[#732265](https://bugzilla.gnome.org/show_bug.cgi?id=732265) and
[#732266](https://bugzilla.gnome.org/show_bug.cgi?id=732266).
