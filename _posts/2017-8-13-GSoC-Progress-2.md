---
layout: post
title: >
    GSoC: One vaapih264dec bug closed, preparing for the final two weeks
---

During the second GSoC coding period I submitted my patches for
[bug 732265](https://bugzilla.gnome.org/show_bug.cgi?id=732265)
and
[bug 732267](https://bugzilla.gnome.org/show_bug.cgi?id=732267).
While the patch for h264parse in 732267 is still under review, my changes in 732265 got accepted and are now in the gstreamer-vaapi [repo](https://cgit.freedesktop.org/gstreamer/gstreamer-vaapi)!
Until the final deadline at August 29 I will be working on
[bug 732266](https://bugzilla.gnome.org/show_bug.cgi?id=732266)
and respond to any new comments in
[bug 732267](https://bugzilla.gnome.org/show_bug.cgi?id=732267).

## Changes for MVC streams in vaapih264dec
With the changes in from 732265 gstreamer-vaapi can now perform the hardware accelerated decoding of H264 MVC streams even if the hardware doesn't support these profiles.
This can be done by setting the new `base-only` vaapih264dec property to `TRUE`.

The patches use the newly created framework for codec-specific properties created after [bug 783588](https://bugzilla.gnome.org/show_bug.cgi?id=783588)
that makes it possible for gstvaapidecode to access the passed value even before the h264 decoder object is created.
If `base-only` is `TRUE` gstvaapidecode adds profiles `multiview-high` and `stereo-high` to the allowed sinkpad caps to make the caps negotiation with the upstream element succeed.

During the decoding process, if a new picture uses a SPS with a `profile_idc` indicating an MVC profile (118 or 128), vaapih264dec will drop the frame.

The changes were tested with all the MVC conformance bitstreams from [ITU-T](https://www.itu.int/net/itu-t/sigdb/spevideo/VideoForm-s.aspx?val=102002641).

## New challenges for SVC streams in vaapih264dec
While working on the final bug, [732266](https://bugzilla.gnome.org/show_bug.cgi?id=732266), I had some new problems that I didn't encounter on MVC streams.
I have to work around the total lack of support for SVC bitstreams in the gstreamer-vaapi codebase and make sure that vaapih264dec doesn't quit because of unsupported units.
Unfortunately, the changes made for 732265 don't work out of the box (returning `GST_VAAPI_DECODER_STATUS_DROP_FRAME` in the `decode_picture` function).
My current strategy is to mark non-Annex-A NAL units as skippable in vaapih264dec's parse function.
This can cause other problems, eg PPS's referring to subset SPS units that are skipped.
Also, a lot of problems seem to be caused because h264parse parses the whole stream, with the non-Annex-A SVC NAL units, and marks access units (AU) differently than with an Annex-A stream where these units are not present.
vaapih264dec has to work around that.

On the up side, all the work for the `base-only` property is ready from
[bug 732265](https://bugzilla.gnome.org/show_bug.cgi?id=732265)
and I have encountered similar issues with my work on
[bug 732267](https://bugzilla.gnome.org/show_bug.cgi?id=732267)
so I am confident that I'll resolve my issues by August 29.

This is it for now, my next post will be part of the final work product submission.
