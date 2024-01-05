# XPSNR Filter Plug-in for FFmpeg

The Extended Perceptually Weighted Peak Signal-to-Noise Ratio XPSNR
is a low-complexity psychovisually motivated distortion measurement
algorithm for assessing the difference between two video streams or
images. This is particularly useful for objectively quantifying the
distortions caused by video or image codecs, as an alternative to a
formal subjective test.  The logarithmic XPSNR output values are in
a similar range as those of traditional PSNR assessments but better
reflect human impressions of visual coding quality. More details on
the XPSNR method can be found in the following scientific papers:

*   C. R. Helmrich, M. Siekmann, S. Becker, S. Bosse, D. Marpe, and
 T. Wiegand, �XPSNR: A Low-Complexity Extension of the Perceptually
 Weighted Peak Signal-to-Noise Ratio for High-Resolution Video Qua-
 lity Assessment,� in Proc. IEEE Int. Conf. Acoustics, Speech, Sig.
 Process. (ICASSP), virt./online, May 2020. www.ecodis.de/xpsnr.htm

*   C. R. Helmrich, S. Bosse, H. Schwarz, D. Marpe, and T. Wiegand,
�A Study of the Extended Perceptually Weighted Peak Signal-to-Noise
 Ratio (XPSNR) for Video Compression with Different Resolutions and
 Bit Depths,� ITU Journal: ICT Discoveries, vol. 3, no. 1, pp. 65 -
 72, May 2020. http://handle.itu.int/11.1002/pub/8153d78b-en

This software allows to determine XPSNR output values, per-frame or
averaged across all assessed frames, between two video streams (the
reference, or input, video and the reconstructed, or output, video)
using the *FFmpeg* software suite, available at https://ffmpeg.org/

When publishing the results of XPSNR assessments (which the license
of this XPSNR implementation allows, see below), a reference to the
above papers as a means of documentation is strongly encouraged.

___________________________________________________________________


Compilation
-----------

This section describes how to compile the source code of this XPSNR
implementation into the FFmpeg executable application as A/V filter
under Linux & macOS.
Please also see https://trac.ffmpeg.org/wiki/CompilationGuide for a
more detailed explanation of the steps required to compile FFmpeg.

First, you need to obtain the latest revision of the *FFmpeg 6.0*
source code from its Git repository:

```
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
```

Then, cd into the directory with the cloned source & reset to the
compatible commit:

```
cd ffmpeg
git reset --hard c5039e158d20e85d4d8a2dee3160533d627b839a
```

Then, you need to copy the files inside the `libavfilter` directory
of this source distribution into FFmpeg's `libavfilter` directory:

`cp xpsnr/libavfilter/* ffmpeg/libavfilter/`

Note that the `allfilters.c` and `Makefile` files already exist in
the FFmpeg source distribution and will be replaced (XPSNR-related
lines have been added in each of these source files). The `xpsnr.h`
and `vf_xpsnr.c` files will be added to the FFmpeg distribution.

Now, while in the `ffmpeg` directory, you can configure and compile
FFmpeg to generate the `ffmpeg` executable with integrated XPSNR
support. Including the `--extra-cflags=-mavx2` flag on compatible x86
CPUs speeds up XPSNR dramatically.

```
./configure --extra-cflags=-mavx2
make
```

If the `--extra-cflags=...` option does not work, you may omit it.


Usage
-----

This section describes how to calculate XPSNR output values between
two video streams using a compiled FFmpeg executable which includes
this plug-in (see the *Compilation* section above). Since the XPSNR
filter works similarly to FFmpeg's existing PSNR filter, it is also
worth taking a look at https://trac.ffmpeg.org/wiki/FilteringGuide,
or http://ffmpeg.org/ffmpeg-filters.html#psnr specifically.

### Simple command-line examples:

The following examples assume an assessment of two 50-Hz input YUVs
(uncompressed video files), inRef.yuv and inTest.yuv, under a Linux
OS. For usage under Windows, replace the `./ffmpeg` by `ffmpeg.exe`.

Two output files are created when using these command-lines, a .log
file with frame-wise XPSNR statistics and a .yuv file (identical to
the inRef.yuv file), which demonstrates that the XPSNR plug-in does
not change the video input. Use of both output files can be omitted
by employing `stats_file=-` instead of `stats_file=(x)psnr.log` and
`-f null -` instead of `-y out.yuv`, respectively.


*8-bit HD reference and test YUVs, first 100 frames:*

`
./ffmpeg -s 1920x1080 -framerate 50 -i inRef.yuv
  -s 1920x1080 -framerate 50 -i inTest.yuv
  -lavfi xpsnr="stats_file=xpsnr.log" -vframes 100 -y out.yuv
`

*10-bit UHD reference and test YUVs, first 100 frames:*

`
./ffmpeg -s 3840x2160 -framerate 50 -pix_fmt yuv420p10le -i inRef.yuv
  -s 3840x2160 -framerate 50 -pix_fmt yuv420p10le -i inTest.yuv
  -lavfi xpsnr="stats_file=xpsnr.log" -vframes 100 -y out.yuv
`

*8-bit HD reference, 10-bit HD test YUV, all frames:*

`
./ffmpeg -s 1920x1080 -framerate 50 -pix_fmt yuv420p -i inRef.yuv
  -s 1920x1080 -framerate 50 -pix_fmt yuv420p10le -i inTest.yuv
  -lavfi xpsnr="stats_file=xpsnr.log" -y out.yuv
`

*8-bit HD reference, 10-bit HD test YUV, PSNR for comparison:*

`
./ffmpeg -s 1920x1080 -framerate 50 -pix_fmt yuv420p -i inRef.yuv
  -s 1920x1080 -framerate 50 -pix_fmt yuv420p10le -i inTest.yuv
  -lavfi psnr="stats_file=psnr.log" -y out.yuv
`


Development
-----------

This fork is not affiliated with the original developers of this
XPSNR FFmpeg implementation. Please disregard the following section
as it is only applicable to the original source.

This section addresses two aspects related to future development of
the XPSNR plug-in for FFmpeg (but not the XPSNR algorithm itself).

### Code contribution:

If you are interested in contributing to this implementation of the
XPSNR algorithm, please contact Fraunhofer HHI or the contributors.
Pull requests with bugfixes and/or speedups are highly appreciated.

### Feature roadmap:

The following functionality is planned to be integrated in a future
revision of this XPSNR implementation. Support is kindly requested.

* support for RGB input (via RGB-to-YCbCr pre-conversion upon read)
* support for multithreading during visual filtering for more speed
* support for metadata as with FFmpeg's existing PSNR, SSIM filters
* direct XPSNR integration into FFmpeg codebase if widely requested


Contact Information
-------------------

Christian Helmrich and Christian Stoffers,
Fraunhofer Heinrich Hertz Institute (HHI),
Video Coding & Analytics (VCA) Department,
Einsteinufer 37, 10587 Berlin, Germany.

For detailed contact information please see �People and Contact� at
https://www.hhi.fraunhofer.de/en/departments/vca.html
