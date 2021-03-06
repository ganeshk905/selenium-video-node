Xvfb Acceleration
=================

This project has some special video acceleration support for Xvfb to improve video encoding performance - current testing shows performance to be at least 6 times faster (so you can save on CPU usage, or run more video nodes).

## Background

A lot of the CPU time used for the video encoding is actually going towards taking the screenshot data using java.awt.Robot - the underlying implementation (for X11) has a lot of nested loops and X11 code to acquire the image (the reason has something to do with having multiple windows, and having to paint each one in the right order, or something). This seems a bit wasteful when using Xvfb, since the screen output is going to a location in memory.

Fortunately, Xvfb has an option which can expose the screen output so that other processes can access it (bypassing the X11 protocol). The Xvfb Acceleration module reads this data directly to grab the screen image, which greatly improves performance.

## Usage

First we have to create a directory for the Xvfb server to write its output into:

    mkdir /tmp/screen

Next we have to modify the xvfb-run command to tell it to write the screen output into the new directory. Modify the xvfb-run command to say:

    xvfb-run -a -s "-screen 0 1280x1024x24 -wr -fbdir /tmp/screen" <video node start command>

Xvfb will memory-map its screen output to a set of files in /tmp/screen, one file per screen attached to that server. For this project we assume that Xvfb only has one screen.

Finally, we have to tell the Selenium Node where to find the screen data. Modify the video node start command to include the option:

    -Dvideo.xvfbscreen=/tmp/screen

The node should then say at startup:

    22:38:00.670 INFO - Using Xvfb acceleration

To say that Xvfb acceleration is enabled.

## Caveats/Gotchas

* Unlike the videos taken using the java.awt.Robot, the cursor will appear in the middle of the screen.
* The bit depth of the Xvfb server MUST be 24 bit colour (8 bits per colour, no alpha). The example command already sets this option.
* The target directory for the Xvfb server must exist, or the server will not start correctly.
* If running multiple Xvfb servers on the same machine, they will need separate target directories. Be careful mapping the java processes to the correct Xvfb server output directories, since its possible to take a video of the wrong Xvfb server!
* Every hour or two of video recording, it seems that the Xvfb server sends a corrupted frame. The code has been designed to detect this and avoid it, but any instances of it breaking videos/crashing the Java process should be reported in the issue tracker.
