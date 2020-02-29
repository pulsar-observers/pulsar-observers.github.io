---
layout: page
type: tutorial
title: Removing RFI From Archival Data
permalink: tutorials/remove-rfi
---

# Removing RFI From Archival Data

To begin data analysis on observation data, we must first download the archive files from the [LWA Pulsar Archive](https://lda10g.alliance.unm.edu/PulsarArchive/). This can be done with two commands, wget, or the alias download. 

First, navigate to your home directory on a UCF node at

```sh
/home/uvastudent/uva_students/<computing ID>
```

Remember to check for an available node at 

http://lwalab.phys.unm.edu/CompScreen/cs.php

The function wget will access and download files from a given web address. For example, to download the 35.1 MHz archive (``.ar``) file on MJD=58835 for pulsar J1929+00, the command line will read

```sh
wget https://lda10g.alliance.unm.edu/PulsarArchive/J1929+00/58835/58835_J1929+00_35.1MHz.ar
```

We can choose to download all four ``.ar`` files by using the wildcard operator ``*`` like so:

```sh
wget https://lda10g.alliance.unm.edu/PulsarArchive/J1929+00/58835/*.ar
```

Better yet, we can use the alias download to retrieve ``.ar`` files from several observations. Use the command

```sh
download --help
```

to review the functions of the alias. 

This alias allows us to input start and end dates for the data download. Continuing from the previous example, if we want to download all ``.ar`` files from MJD=58531 to MJD=58835, type

```sh
download J1929+00 --start_date=58531 --end_date=58835
```

Checking the Pulsar Archive website, this command spans three observations, or 12 ``.ar`` files. While using download is an efficient way to retrieve many data files, each file is on the order of 1 GB in size, so do not download many (8+) files at once. 

Now, with your ``.ar`` files downloaded, we will perform pscrunch on the files, which removes unneeded polarization data from the ``.ar`` file.

```sh
pam -p -e pscrunch *.ar
```

Using the wildcard operator, we can scrunch all of the data at once. 

Once the data has been scrunched, we can begin zapping data. This allows the data to be visualized in an array, and eliminating RFI is easier to interpret. 

We can open a ``.scrunch`` file with the command

```sh
psrzap 58835_J1929+00_35.1MHz.scrunch
```

This command will bring up two PGPLOT windows, shown below:

The window with three graphs displays: 1) the pulse profile, 2) frequency vs. pulse phase, 3) time vs. pulse phase. As we eliminate RFI, the pulse profile should become more defined. 

![pgplots](https://github.com/pulsar-observers/pulsar-observers.github.io/blob/master/assets/img/pgplot1.PNG)


The other window with a single plot displays the data as a visual array of the data, organized by frequency and sub-integration index. We will be working with this plot to visually eliminate the RFI in our data. Bright spots in the data represent RFI, which we want to eliminate. Each time a pixel of RFI is eliminated, the plot will recalibrate its display values. 

Sometimes, RFI is so strong that the plot looks empty save for a few rectangles. Once these are elimnated, the rest of the data will become visible. 

Clicking into the single plot, we will use an array of commands to "zap" the RFI. 

r - "reset" - rescales the zoom to display the whole plot, but will not reset the plot to the original state

t - "time" - changes the pointer to a vertical line

f - "frequency" - changes the pointer to a horizontal line

b - "both" - changes the pointer to crosshairs (default)

u - "undo" - will undo the last "zap" in case the wrong pixel was selected

s - "save" - will save the file and all your progress

q - "quit" - allows the user to safely exit the program 

To mark a pixel, row, column, or rectangle, right click the selection, and type 'r' to eliminate the RFI. To zoom into a section, left click to establish the left bound, then 













