---
layout: page
type: tutorial
title: LWA Pulsar Data Reduction Tutorial
hide: true
permalink: tutorials/LWA-data-reduction
---

# LWA Pulsar Data Reduction Tutorial

Accompanying lecture: [LWA Pulsar Data Reduction]({{ site.baseurl }}/lectures/LWA-data-reduction)

### Intro
LWA Pulsar data is reduced to a large number of data products with the intent to allow for us to do whatever reduction we wish to do and also to allow any future pulsar astronomers who grab the data from the LWA Pulsar Data Archive to do whatever it is they want to do. We reduce the data in such a way that there are files that support the following pulsar science:

  - Pulsar Timing
  - Polarization studies
  - Analysis of individual pulses
  - Pulse Profile vs frequency/time/whatever
  - Flux density measurements
  - Study of low frequency radio frequency interference


A typical LWA pulsar observation results in about 130 files that are useful for some sort of pulsar study. Of course, this is a fairly complicated process, but luckily the hard work has already been done to develop the various tools needed.

### Checking a file for "timetag errors"
A station of the LWA consists of ~256 dipole antennas each with two polarizations for a total of 512 signals. These signals go into an analog receiver (ARX or ASP) and then into a digital processor (DP) which then sends the data into "Data Recorders" where the signal is written to a set of disks. If there is a slow down in DP or when writing the disks, there can be a break in the data. In order to know whether or not such a break has occurred, the data are recorded in pieces that we call "frames". Each frame has a time tag recorded with it to let us know what time the data corresponds to. If there is a break in the data, we can tell from these time tags and we call these "timetag errors". So, before getting into the rest of the data analysis, it is generally a good idea to check for these timetag errors at the beginning. This can be done using the command:

```sh
$ /home/jdowell/fastDRXCheck.py FILENAME
```

It would be useful to look at a case for a good file and for a bad one so that you know what you are looking for. Okay, so first a good file:

```sh
$ /home/jdowell/fastDRXCheck.py /data/network/recent_data/uvastudent/J1929+00/058043_000733414
Valid Byte Ranges:
             0, 569678328288 -> 530.554 GB or 138.003 Mframes
-> 100.0% contiguous in 4096 frame blocks
```

This says that the file goes from byte 0 to byte 569678328288 without any errors, so the file seems to not have any timetag issues. Now lets take a look at a file with one timetag error:
```sh
$ /home/jdowell/fastDRXCheck.py /data/network/recent_data/uvastudent/J1929+00/058043_000733413
Valid Byte Ranges:
             0,  90751227552 ->  84.519 GB or  21.984 Mframes
   90785997696, 569681316960 -> 446.006 GB or 116.011 Mframes
-> 100.0% contiguous in 4096 frame blocks
```
You can see that the output now has two lines and that there seems to be an error about 84.5 GB into the file. If you get a timetag error, typically the easiest thing to do is just split the file and process the pieces independently. You will have to let me know that you have timetag errors and I will split the file for you, because your account does not have write access to the directory in which the raw data is kept.

### "Simple" LWA data reduction

As I mentioned earlier, the full processing of LWA pulsar data can be complicated, but fortunately the hard work has been done previosly and if everything goes right, there are just two simple commands that are necessary.

```sh
$ /home/kstovall/bin/makereducepulsar.py PSRNAME 0/1 FILENAMES > Make.psr
$ make -j6 -l10 -f Make.psr > reducepulsar.out &
```

So, what do these commands do? First, let's take a look at:

```sh
$ /home/kstovall/bin/makereducepulsar.py PSRNAME 0/1 FILENAMES > Make.psr
```
This is "simply" a script written in Python that takes the names of the LWA raw data files that you supply (FILENAMES) and generates a GNU Makefile (which we redirected to the file named Make.psr) that can be used to generate all of the reduced data products for the specified pulsar (PSRNAME) to be put in the LWA Pulsar Archive. Some of these data products are what you will be using to "time" your pulsar. The "0" or "1" that is supplied as the second argument to the script tells it either to combine the data from the two files ("1") or not ("0"). Typically, pulsar observations are done using 2 beams and if you have both files and they are free from timetag errors, then this should be a "1".

Behind the scenes, there is quite a bit going on, first the script will look up information about the pulsar that you gave it, which is stored in a database. This database stores the best DM for this pulsar as well as various parameters that are optimal for the pulsar based on its spin period, DM, and how bright it is. Then it will output a "makefile". A makefile is a set of rules (or recipes) that tells GNU "make" how to get from input files to a set of files that are required at the end. Many of the commands can depend on other commands within the makefile. With a properly made makefile, GNU make will sort out all of these dependencies and run them in the correct order. The simplest make command, which tells it to just make all of the files is:

```sh
$ make -f Make.psr
```
However, this would run every command one at a time. We would like to take advantage of as much of the computing resources as we can (though we don't want to end up trying to use too much). So, we add in the -j and -l options. The -j option tells make how many "threads" to use, which is essentially how many commands to keep running at once. The -l option is used in order to try to prevent make from using too many resources. It means to only start up new commands if the "load" on the computer is below the given value. So, in the make command I gave above, it would keep 6 commands running (assuming there are commands available to run) and would only run additional commands if the load is below 10.

### What files are made and what commands are being run to make them?
Before getting into the commands, it would be useful to know how to see the various parameters that are used for a particular pulsar. Above, I said this is stored in a database, but I also have found that it is useful to keep a text file (/home/kstovall/LD002_DMDB.txt) of the parameters as well. To get the parameters, you can just use the `grep` command. Below, I will give some command examples for reducing data for the pulsar B0136+56, so lets have a look at its parameters:
```sh
$ grep B0136+57 ~/LD002_DMDB.txt 
B0136+57   73.81141  4096 01:39:19.7401     +58:14:31.819       64     512     128    2048    60      1024    2048 60 16384 8 0
```
These all give information for how the data should be processed. There are quite a few parameters, so I won't go into what they all mean but wanted to show you how you can have a look at them if interested. You will see many of these parameters come up in example commands below.

Most of the commands that the make file runs are from standard pulsar data reduction packages, `PRESTO`, `DSPSR`, and `PSRCHIVE`. However, these packages don't understand raw LWA data, so there are two commands that are used as intermediate steps to get the data into a format that standard pulsar software can understand. These are commands included in the Pulsar extension of the LWA Software Library (LSL), which can be found at https://fornax.phys.unm.edu/lwa/trac/wiki. These commands are `drx2drxi.py` and `writePsrfits2DMulti.py`. Let's start with the simpler one, drx2drxi.py.

##### `drx2drxi.py` and `dspsr`
The raw LWA data format is actually very similar to a format known as Mark5C that is used (or at least has been used) at many other radio telescopes and can be interpreted by the DSPSR command. The only issue is that there are two tunings in a single file, which is typically not the case for Mark5C. So, drx2drxi.py reads in the raw data and outputs two files, one containing the data for each tuning. Here is an example of how to run it:
```sh
$ /usr/local/extensions/Pulsar/drx2drxi.py /data/network/recent_data/kstovall/LS006/B0136+57/058036_000662236
```

Running this command results in two new files in your current directory: 058036_000662236_b4t1.dat and 058036_000662236_b4t2.dat

These can then be fed into the `dspsr` command, which coherently dedisperses, channelizes, and folds it using an "ephemeris" for the pulsar. An ephemeris is a file containing the position, DM, spin, and binary (if applicable) for the given pulsar. If you want to have a look at an ephemeris file (also known as a par file), there are a lot in /home/kstovall/tzpar/, including one for the pulsar you are working on. Here is an example `dspsr` command.
```sh
$ dspsr -F2048:D -b1024 -E /home/kstovall/tzpar/B0136+57.par -L60 -A -O 58036_B0136+57_64.5MHz -a PSRFITS -minram=256 -t1 B0136+57_64.5.hdr
```
 There are quite a few arguments here, but here is a description of each:
 `-F2048:D` means to coherenetly dedisperse and channelize the data into 2048 channels
 `-b1024` means to use 1024 bins across the profile of the pulsar
 `-E /home/kstovall/tzpar/B0136+57.par` tells it what ephemeris to use
 `-L60` tells it to fold for 60 seconds and write that chunk of data to disk
 `-A` means to output as only one file
 `-O 58036_B0136+57_64.5MHz` tells it the output file name (it adds an extension of .ar)
 `-a PSRFITS` tells it to use PSRFITS (explained more below) as the output format
 `-t1` says to use one CPU thread
 `B0136+57_64.5.hdr` is the name of a file (made by the makefile) that contains information on the pulsar and the name of the data file.
 
 This dspsr command results in a file `58036_B0136+57_64.5MHz.ar` that can now be used by the `PSRCHIVE` suite of pulsar data reduction tools. We will use some of these later, but the .ar file is what goes into the LWA Pulsar Archive, so that is all we need right now.

##### `writePsrfits2DMulti.py` and `combine_lwa2`
There are actually a set of scripts that this one is a part of, `writePsrfits2.py`, `writePsrfits2D.py`, `writePsrfits2Multi.py`, and `writePsrfits2DMulti.py`. Let's start with a description of `writePsrfits2.py`, this reads in the raw LWA format, splits the two tunings, channelizes the data, and outputs the resulting data in what is known as PSRFITS format (NOTE: It does not fold the data). PSRFITS is a modification of the (Flexible Image Transport System) FITS format standard to support pulsar data. This data format is understood by many pulsar data reduction packages, including `PRESTO` which is the one that we will use. Now, the `writePsrfits2D.py` tool does the same thing, except during the channelization step, it also coherently dedisperses within each channel. The tools with Multi in the name do the same things as these, but can be given raw files from multiple beams that were observed simultaneously. These have the added step of looking at the start and end of each file and only outputting the data from the period when all supplied beams were actually observing. So, if one beam began observing a little before the others, it would skip that data. If our files don't have timetags, use `writePsrfits2DMulti.py` so that we can easily combine both beams. If there are timetag errors, you likely want to use 
`writePsrfits2D.py` for each file. So, basically what happens within the makefile described before, if that second argument is a 0, it uses `writePsrfits2D.py` and if the second argument is a 1, it uses `writePsrfits2DMulti.py`. Here is an example usage of `writePsrfits2DMulti.py`
```sh
$ /usr/local/extensions/Pulsar/writePsrfits2DMulti.py --source=B0136+57 --ra=01:39:19.7401 --dec=+58:14:31.819 --nchan=4096 --nsblk=16384 73.81141 /data/network/recent_data/kstovall/LS006/B0136+57/058036_000662236 /data/network/recent_data/kstovall/LS006/B0136+57/058036_000662237
```
Again, there are quite a few arguments, but here is a description of each:
`--source=B0136+57` tells it the name of our pulsar which gets put into the PSRFITS header information
`--ra=01:39:19.7401 --dec=+58:14:31.819` are the right ascension and declination of the pointing
`--nchan=4096` tells it how many frequency channels to divide the data into
`--nsblk=16384` tells it how many time bins to put into a "block" of data
`73.81141` is the DM used for coherent dedispersion
and then you have to specify the names of the raw data files that are being processed.

Note that this can take many hours to run. Running this command would result in the creation of 4 new files:
```sh
$ ls -lha *.fits
-rw-rw-r-- 1 kstovall kstovall 132G Oct 12 08:19 drx_58036_B0136+57_b3t1_0001.fits
-rw-rw-r-- 1 kstovall kstovall 132G Oct 12 08:19 drx_58036_B0136+57_b3t2_0001.fits
-rw-rw-r-- 1 kstovall kstovall 132G Oct 11 22:56 drx_58036_B0136+57_b4t1_0001.fits
-rw-rw-r-- 1 kstovall kstovall 132G Oct 11 22:56 drx_58036_B0136+57_b4t2_0001.fits
```
These can then be combined into a file using a tool written specifically for this that is called `combine_lwa2`. It would need to be run 3 times, once for each beam to combine the tunings and then one more time to combine the two beams. So, running it would look something like:
```sh
$ /home/kstovall/src/psrfits_utils_scott_combinetest/combine_lwa2 -o drx_58036_B0136+57_b4 drx_58036_B0136+57_b4t2_0001.fits drx_58036_B0136+57_b4t1_0001.fits
```
Which would result in a new file `drx_58036_B0136+57_b4_0001.fits`
```sh
$ /home/kstovall/src/psrfits_utils_scott_combinetest/combine_lwa2 -o drx_58036_B0136+57_b3 drx_58036_B0136+57_b3t2_0001.fits drx_58036_B0136+57_b3t1_0001.fits
```
Which would result in a new file `drx_58036_B0136+57_b3_0001.fits`
```sh
$ /home/kstovall/src/psrfits_utils_scott_combinetest/combine_lwa2 -o drx_58036_B0136+57 drx_58036_B0136+57_b4_0001.fits drx_58036_B0136+57_b3_0001.fits
```
Which would result in the new file `drx_58036_B0136+57_0001.fits`. At this point, we no longer need the two combined beams. For future steps, we only use the .fits files from the individual tunings and the one fully combined one:
`drx_58036_B0136+57_b3t1_0001.fits`, `drx_58036_B0136+57_b3t2_0001.fits`, `drx_58036_B0136+57_b4t1_0001.fits`, `drx_58036_B0136+57_b4t2_0001.fits`, `drx_58036_B0136+57_0001.fits`
Okay, now we have the PSRFITS files, which can be processed using `PRESTO` commands. 

##### PRESTO commands
The remainder of the commands involve running various tools provided by `PRESTO`. In pulsar data analysis (and actually all of radio astronomy), a interference from earth and satellite radio signals can be a big problem. These are generally referred to as radio frequency interference or RFI. Our next step is to identify RFI and create a "mask" for the data, which can be used to tell subsequent data analysis tools which parts of the data should just be ignored. This is done using `rfifind` by doing something like the following:

```sh
rfifind -time 10 -o drx_58036_B0136+57_b4t1 drx_58036_B0136+57_b4t1_0001.fits
```
`-time 10` means to read in ~10 seconds of data at a time to look for RFI
`-o drx_58036_B0136+57_b4t1` tells it what to use as the basename for output
This command results in the creation of 5 files (there is a 6th below because I redirect the output to `drx_58036_B0136+57_b4t1_rfifind.out` when I run it).
```sh
$ ls *_b4t1_rfifind*
drx_58036_B0136+57_b4t1_rfifind.inf   drx_58036_B0136+57_b4t1_rfifind.ps
drx_58036_B0136+57_b4t1_rfifind.mask  drx_58036_B0136+57_b4t1_rfifind.rfi
drx_58036_B0136+57_b4t1_rfifind.out   drx_58036_B0136+57_b4t1_rfifind.stats
```
Most of these files are just binary or test information about the RFI in the file, but the .ps file is an image that can be looked at to graphically represent when and at what frequencies the RFI occurs.

Now that we have generated the RFI mask files, we can move on to generating other data products. Note that in what follows, I am showing what is done for 1 of the PSRFITS files above, but all of these steps are done for all 5 of them.

##### `prepfold`
The next step is to fold the data using the prepfold command. Here is an example of its usage:
```sh
$ prepfold -ncpus 2 -mask drx_58036_B0136+57_b4t1_rfifind.mask -timing /home/kstovall/tzpar/B0136+57.par -n 64 -nsub 128 -noxwin -o drx_58036_B0136+57_b4t1_timing_masked -dm 73.81141 drx_58036_B0136+57_b4t1_0001.fits
```
`-ncpus 2` tells it to use 2 CPUs (for OpenMP)
`-mask drx_58036_B0136+57_b4t1_rfifind.mask` tells it to mask sections of data based on the mask we generated earlier
`-n 64` tells it to use 64 bins across the pulse profile
`-nsub 128` tells it to use 128 frequency channels
`-noxwin` by default, it opens a window with an image of the resulting fold, this tells it not to try to do that
`-o drx_58036_B0136+57_b4t1_timing_masked` specifies the base name for output files
`-dm 73.81141` tells it the DM for incoherent dedispersion
and then the filename

prepfold has the ability to search a little bit in period, period derivative, and DM. For the Pulsar Archive, we generate 4 different folds, one that does not allow for it to search which is shown above (the -timing argument tells it not to search, but to just fold exactly at the ephemeris information), one that allows for it to search a little bit in case the period, DM, etc. from the ephemeris is a little bit off (substitute -search in for -timing) and then these are each repeated but without a RFI mask supplied. So, we would also do:
```sh
$ prepfold -ncpus 2 -mask drx_58036_B0136+57_b4t1_rfifind.mask -search /home/kstovall/tzpar/B0136+57.par -n 64 -nsub 128 -noxwin -o drx_58036_B0136+57_b4t1_search_masked -dm 73.81141 drx_58036_B0136+57_b4t1_0001.fits
```
```sh
$ prepfold -ncpus 2 -timing /home/kstovall/tzpar/B0136+57.par -n 64 -nsub 128 -noxwin -o drx_58036_B0136+57_b4t1_timing_unmasked -dm 73.81141 drx_58036_B0136+57_b4t1_0001.fits
```
```sh
$ prepfold -ncpus 2 -search /home/kstovall/tzpar/B0136+57.par -n 64 -nsub 128 -noxwin -o drx_58036_B0136+57_b4t1_search_unmasked -dm 73.81141 drx_58036_B0136+57_b4t1_0001.fits
```

##### `prepdata`
We now want to make a file that consists of just a time series that has been dedispersed at the pulsar's DM. These can be useful for looking at the behavior of individual pulses. We do this with and without supplying the RFI mask, so we run the command shown below with and withpout the -mask argument.
```sh
prepdata -ncpus 2 -dm 73.81141 -mask drx_58036_B0136+57_b4t1_rfifind.mask -nobary -o drx_58036_B0136+57_b4t1_DM73.81141_topo_masked drx_58036_B0136+57_b4t1_0001.fits
```
The only new argument here is the `-nobary` flag. By default, prepdata will adjust the data such that the data is as if it was taken at the "solar system barycenter". The `-nobary` flag tells it not to do that. Instead, it will not be adjusted and will be left in topocentric coordinates, which is a frame of reference travelling with the observatory.

Something that can cause problems if you are just looking at the dedispersed timeseries made above is distinguishing between pulsar and RFI. So it is often good to know what is going on at the same time in the timeseries at DM=0. So, we also make that with and without the mask. So, this just means running the same command, but with `-dm 0`. This data can also be useful just for looking at the RFI environment and could be used to monitor low frequency RFI over time.
That is the end of the `PRESTO` commands, but we have one more command to run.

##### `psrfits_subband`
We have now generated most of the files that we want. We would like to just keep the PSRFITS files that we have, but they are very large (in the example I have been using, they take up about 1TB of space). We can keep a file that is similar, but we have to give up some frequency and time resolution. So, we make such a file using `psrfits_subband` with a command like:
```sh
$ /home/kstovall/src/psrfits_utils_scott/psrfits_subband -weights drx_58036_B0136+57_b4t1_rfifind.weights -o drx_58036_B0136+57_b4t1_subs -nsub 128 -dm 73.81141 -dstime 8 drx_58036_B0136+57_b4t1
```
`-weights drx_58036_B0136+57_b4t1_rfifind.weights` tells it how to weight (0 or 1) each frequency channel based on the earlier RFI masking, drx_58036_B0136+57_b4t1_rfifind.weights is a file made from running drx_58036_B0136+57_b4t1_rfifind.mask through a script at /home/kstovall/bin/rfifind.py. Basically, a channel with lots of RFI will not be used.
`-o drx_58036_B0136+57_b4t1_subs` specifies the basename for the output file
`-nsub 128` tells it to reduce to 128 frequency channels
`-dm 73.81141` the channels are incoherently dedispersed at this DM before being combined
`-dstime 8` tells it to reduce the time resolution by a factor of 8 (i.e. add 8 adjacent time bins together)
The result is a file called `drx_58036_B0136+57_b4t1_subs_0001.fits` which is a factor of 256 smaller in size (8 in time and 32 in frequency).
