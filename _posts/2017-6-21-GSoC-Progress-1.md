---
layout: post
title: My progress during the first weeks of GSoC
---

As days pass and GSoC's first coding period deadline is closing by, I thought I should update my blog with my progress so far.

## Dropping the non-Annex-A units from vaapih264dec's side

### The base-only property

The first issue I tried to tackle was the addition of a boolean property to the `vaapih264dec` plugin.
When `base-only=TRUE` vaapih264dec should:

1. Allow the negotiation of MVC profiles with upstream elements, even if the hardware does not support them.
2. Drop any non-Annex-A units from the stream.

Unfortunately, unlike the encoders of gstreamer-vaapi, there isn't a framework in place for adding codec-specific properties to a vaapidecoder.
This limitation also affects the unrelated bug
[#762509](https://bugzilla.gnome.org/show_bug.cgi?id=762509).
Matt Staples, to whom the bug is assigned, had a patch ready before I could look further into the issue and new
[bug report](https://bugzilla.gnome.org/show_bug.cgi?id=783588)
was created.

The patches that will be finally accepted for the latter bug will define the final resolution of my assignments
[#732265](https://bugzilla.gnome.org/show_bug.cgi?id=732265) and [#732266](https://bugzilla.gnome.org/show_bug.cgi?id=732266)
but for the time being I am using Matt's
[attachment 353187](https://bugzilla.gnome.org/attachment.cgi?id=353187)
to add and read the `base-only` property.

### Dropping the units

The units that vaapih264dec should drop to confront to Annex-A profiles are mentioned in the
[h264 spec](https://www.itu.int/rec/T-REC-H.264):

> Decoders that conform to one or more of the profiles specified in Annex A rather than the profiles specified in Annexes G or H shall ignore (remove from the bitstream and discard) the contents of all NAL units with nal_unit_type equal to 14, 15, or 20.

In gstreamer-vaapi's `gstvaapidecoder_h264.c` file these unit types correspond to values `GST_H264_NAL_PREFIX_UNIT`, `GST_H264_NAL_SUBSET_SPS` and `GST_H264_NAL_SLICE_EXT`.
Alternatively, an MVC unit can be detected with the function `is_mvc_profile` through the corresponding sps object
(sps stands for *Sequence Parameter Set*, a NAL unit containing parameters that apply to a series of consecutive coded video pictures).

I've found that a good place to drop these units from the stream is by returning `GST_VAAPI_DECODER_STATUS_DROP_FRAME` in the `decode_picture` function.

### Modifying caps negotiation

The current behaviour of the generic gstvaapidecoder during caps negotiation is to include in its
[allowed sinkpad caps](https://gstreamer.freedesktop.org/documentation/application-development/basics/pads.html#capabilities-of-a-pad)
only the profiles supported by the hardware, which can be found through the `vainfo` command.
When the `base-only` property is active we'd like for gstvaapidecoder to "lie" to upstream elements about the allowed caps so that the pipeline can be created successfully.

This was achieved by modifying `gstvaapidecode.c`'s `gst_vaapidecode_ensure_allowed_sinkpad_caps` function.
The profiles "multiview-high" and "stereo-high" are forcefully added when the following conditions hold:

1. The `base-only` property is activated.
2. Annex-A high profiles are available.
3. MVC profiles are not available.

One challenge faced here is accessing the `base-only` property before the actual h264 decoder object is initialized.

### Results & patch

I have tested my changes to gstreamer-vaapi's codebase mainly with MVC sample files provided by
[ITU](https://www.itu.int/net/itu-t/sigdb/spevideo/VideoForm-s.aspx?val=102002641).
I use a simple script to check the size differences with and without the `base-only` property between the raw, decoded streams and the differences in execution times:

```
#!/bin/bash
set -e

function myexec() {
    local file="$1"
    local base_only="$2"
    >&2 echo "Running for $file, base-only=$base_only"

    local cmd="gst-launch-1.0 -q filesrc location=$file ! h264parse ! vaapih264dec base-only=$base_only ! filesink location=tmp.raw"
    rm -f tmp.raw

    local ts=$(date +%s%N)
    eval "$cmd"
    local tt=$(($(date +%s%N) - $ts))
    echo "$tt"
}

trap "rm -f tmp.raw" EXIT
trap "rm -f tmp.raw" SIGINT
trap "rm -f tmp.raw" SIGTERM

for f in $(ls *.264); do
    time_true=$(myexec "$f" TRUE)
    size_true=$(wc -c < tmp.raw)

    time_false=$(myexec "$f" FALSE)
    size_false=$(wc -c < tmp.raw)

    size_frac=$(echo "100 * $size_true / $size_false" | bc -l)
    time_frac=$(echo "100 * $time_true / $time_false" | bc -l)
    printf "Size diff: %10d / %10d = %.2f%%\n" $size_true $size_false $size_frac
    printf "Time diff: %10d / %10d = %.2f%%\n" $time_true $time_false $time_frac
    echo -------------------------------------------
done
```

I have submitted a [preliminary patch](https://bugzilla.gnome.org/attachment.cgi?id=354073&action=diff) for
[#732265](https://bugzilla.gnome.org/show_bug.cgi?id=732265).
It still needs to be properly reviewed and there is still the blocking bug
[#783588](https://bugzilla.gnome.org/show_bug.cgi?id=783588)
mentioned earlier that hasn't been resolved.

## Dropping the non-Annex-A units from h264parse's side

From h264parse's side when a downstream element only reports Annex-A profiles
(if, for example, the hardware doesn't support MVC decoding like in vaapih264dec case)
but we are parsing an MVC encoded stream, it is desirable to only parse the base view of the stream, dropping non-Annex-A NAL units.

The logic I follow to implement this is:

1. `get_compatible_profile_caps()`: "Mark" the `"main"` and `"high"` profiles as compatible to MVC profiles.
2. `ensure_caps_profile()`: Initialize the `base_only` variable to `TRUE` if sps' `profile_idc` shows an MVC profile and an Annex-A profile has been negotiated.
3. `gst_h264_parse_handle_frame()`: If `base_only==TRUE`, don't process all NAL units with type 14, 15 or 20.

My changes are available in my
[fork repo](https://github.com/orestisf1993/gst-plugins-bad/commit/4f1e480ab2921752a0bf1c8b06557d8717bd8217)
but I haven't submitted a patch in bug
[#732267](https://bugzilla.gnome.org/show_bug.cgi?id=732267)
yet since I want to investigate the situation with packetized streams more.

A way to check the changes using the vaapih264dec decoder is to remove from `gstvaapiprofile.c` the MVC profiles from `gst_vaapi_profiles[]`:

```
diff --git a/gst-libs/gst/vaapi/gstvaapiprofile.c b/gst-libs/gst/vaapi/gstvaapiprofile.c
index 9c353c56..c30f8733 100644
--- a/gst-libs/gst/vaapi/gstvaapiprofile.c
+++ b/gst-libs/gst/vaapi/gstvaapiprofile.c
@@ -106,10 +106,10 @@ static const GstVaapiProfileMap gst_vaapi_profiles[] = {
   {GST_VAAPI_PROFILE_H264_HIGH, VAProfileH264High,
       "video/x-h264", "high"},
 #if VA_CHECK_VERSION(0,35,2)
-  {GST_VAAPI_PROFILE_H264_MULTIVIEW_HIGH, VAProfileH264MultiviewHigh,
-      "video/x-h264", "multiview-high"},
-  {GST_VAAPI_PROFILE_H264_STEREO_HIGH, VAProfileH264StereoHigh,
-      "video/x-h264", "stereo-high"},
 #endif
   {GST_VAAPI_PROFILE_VC1_SIMPLE, VAProfileVC1Simple,
       "video/x-wmv, wmvversion=3", "simple"},
```
and then run some pipelines without the `base-only` property.
