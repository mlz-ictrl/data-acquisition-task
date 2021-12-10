## Intro

In neutron scattering, we shoot neutrons (tiny elementary particles) onto a
substance and want to know if they change direction.  Therefore, a common type
of detector is, like a computer screen, divided into many pixels.  When it is
placed behind the substance, it can measure when and where a neutron hits one of
the pixels.

This task simulates performing data acquisition: receiving data from a detector
over the network, with a format given by the manufacturer, parsing it and making
a quick visualization of the data.

In addition to this specification, the repository contains two .data files that
represent data streams from the detector, and for each one a .png image that
shows the desired final result.

## Your task

- Write a very simple server in C that accepts a TCP connection on port 50000
  and simply streams the contents of a dump file to a client.

  It does not need to handle multiple clients, no threading is required.

- Write a client in Python that connects to the server, receives the data and
  parses it according to the format documented below.

  It should create a histogram of the data using numpy (see below for an
  explanation) and after reaching the end of the data, display the histogram as
  an image using matplotlib.

All code must run on a Linux host running e.g. Ubuntu 20.04.  It should handle
unexpected errors (like incomplete packets or a broken TCP connection) reasonably.
Don't use any libraries except for the C and Python standard libraries, numpy and
matplotlib.  The code does not need to be particularly optimized for speed.

## Data format specification

The file content is a sequence of *packets*.  Each packet contains a *header*
and optionally a number of *events*.

The *header* is 16 bytes long and has the following structure:

```
+---------+---------+-------------------+
| length  | pktype  | timestamp         |
| 32 bits | 32 bits | 64 bits           |
+---------+---------+-------------------+
```

All integers are in little-endian format:

- The `length` gives the length of the *whole* packet in bytes.

- If `pktype` is anything other than `0x1000`, the data does not contain useful
  events and must be discarded.

- The `timestamp` can be ignored for your task.

If `pktype` is `0x1000`, the rest of the packet contains a number of *events*,
each of which is a 32-bit little-endian integer:

```
MSB                               LSB
+---------+---------+---------------+
| X coord | Y coord | timestamp     |
| 7 bits  | 7 bits  | 18 bits       |
+---------+---------+---------------+
```

`X` and `Y` are the pixel coordinates used for the histogram, the `timestamp`
can again be ignored.

## X/Y values and histogramming

The client should create an X-Y histogram of the events, which means the
following:

- At the start, create a new 2-dimensional numpy array filled with zeros and the
  dimensions 128x128 (enough for 7-bit coordinates).

- Then, when you process each event, add 1 to the array element at the event's X
  and Y coordinate.

- In the end, if you display the array as an image using matplotlib, you should
  get something that looks like the .png file in this repository.
