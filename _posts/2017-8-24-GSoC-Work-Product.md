---
layout: post
title: My work product submission for GSoC 2017
---

## Work done
With my gstreamer-vaapi patches, the vaapih264dec plugin can perform the hardware accelerated decoding of H264 MVC / SVC streams even if there is no hardware support for the stream's profile.
This can be done by enabling the `base-only` property of the element which will make vaapih264dec drop the non-base view frames.
This should make all SVC streams at least playable, something that vaapih264dec, and most well-known open source media player applications, didn't support so far.

My patches for h264parse add a similar feature:
If a downstream element doesn't support MVC or SVC profiles but such a stream is detected, h264parse will drop the NAL units related to non-base views, and pass only the H264 Annex-A compatible substream to the downstream element.
This feature can be used with every gstreamer h264 decoder element as long as it reports its supported profiles in its caps.

## Submitted patches
My patches can be found in the attachments of the bug report pages on the gnome bugzilla:
- Bug 732265 - vaapidecode: h264: add support for decoding MVC base views only:  
[https://bugzilla.gnome.org/show_bug.cgi?id=732265](https://bugzilla.gnome.org/show_bug.cgi?id=732265)
- Bug 732266 - vaapidecode: h264: add support for decoding SVC base layers only:  
[https://bugzilla.gnome.org/show_bug.cgi?id=732266](https://bugzilla.gnome.org/show_bug.cgi?id=732266)
- Bug 732267 - h264parse: extract base stream from MVC or SVC encoded streams:  
[https://bugzilla.gnome.org/show_bug.cgi?id=732267](https://bugzilla.gnome.org/show_bug.cgi?id=732267)

## Merged commits
Can be viewed in the following links:
- For gstreamer/gstreamer-vaapi:  
[https://cgit.freedesktop.org/gstreamer/gstreamer-vaapi/log/?qt=author&q=orestis](https://cgit.freedesktop.org/gstreamer/gstreamer-vaapi/log/?qt=author&q=orestis)
- For gstreamer/gst-plugins-bad:  
[https://cgit.freedesktop.org/gstreamer/gst-plugins-bad/log/?qt=author&q=orestis](https://cgit.freedesktop.org/gstreamer/gst-plugins-bad/log/?qt=author&q=orestis)

## Other bug reports
During the GSoC coding periods I also submitted the following bug reports for problems detected while implementing my original task:
- [https://bugzilla.gnome.org/show_bug.cgi?id=783588](https://bugzilla.gnome.org/show_bug.cgi?id=783588)
- [https://bugzilla.gnome.org/show_bug.cgi?id=786797](https://bugzilla.gnome.org/show_bug.cgi?id=786797)
- [https://bugzilla.gnome.org/show_bug.cgi?id=786802](https://bugzilla.gnome.org/show_bug.cgi?id=786802)

## Work to be done
- Get the code accepted: bugs 732266 and 732267 are still open and my patches are under review.
I will reply to reviews and comments until they get accepted.
- Filter out SEI messages according to the H264 spec.
This is optional for the decoding process but still desired.
I am interested in submiting a patch for this.

## Other useful links
- [GSoC project page](https://summerofcode.withgoogle.com/projects/#4983264524632064)
- Blog I used during GSoC: [https://orestisf1993.github.io/](https://orestisf1993.github.io/)
- My gnome bugzilla profile: [https://bugzilla.gnome.org/page.cgi?id=describeuser.html&login=orestisf1993%40gmail.com](https://bugzilla.gnome.org/page.cgi?id=describeuser.html&login=orestisf1993%40gmail.com)
