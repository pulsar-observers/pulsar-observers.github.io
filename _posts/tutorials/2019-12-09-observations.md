---
layout: page
type: tutorial
title: Creating and Scheduling Observation Files
permalink: tutorials/make-observations
---

# Creating and Scheduling Observation Files

### Introduction

Though the LWA Pulsar Archive is full of past observation data that needs to be processed, we will also be creating our own new observations to add to this data collection. When we create our timing solutions, the each observation creates a point on the residuals diagram; by scheduling new observations, we are adding a new and recent data point to the timing solution. 

### Creating Observation Files

Follow the regular steps to log onto the LWA computers. Log onto a UCF after validating its availability at [LWAUCF Operations](http://lwalab.phys.unm.edu/CompScreen/cs.php). For example, to log onto ``ucf1``, use the ``ssh`` shortcut:

```sh
$ ucf1
```

We will now create our session definition files for observing. More information can be found at the [LWA observing website](http://www.phys.unm.edu/~lwa/astro/scheds/schedhints.html). Use the shortcut 

```sh
$ sessionmaker
```

to replace the command

```sh
 /usr/local/extensions/SessionSchedules/sessionGUI.py
```

This pulls up a GUI for creating observation files. 
Go to **File** > **New** and fill in the following information:

##### Observer Information
ID Number: 55

First Name: Steven

Last Name: Stetzler
##### Project Information 
ID Code: LS012

Title: <pulsar name>
	
Comments: none (leave blank)
##### Session Information
ID Number: 190_<internal count 1,2,3,..>		
Title: <pulsar name>
Comments: none (leave blank)

Session Type - select BEAM FORMING
Data Return Method - select COPY TO UCF
UCF Username: uvastudent

And then click **Ok**

Go to **Observations** > **Add** > **DRX-RA/DEC** and fill in across the row:
Name: <pulsar name>
Target: <pulsar name>
Comments: none
Start: Set the duration to 24h with ``24:00:00``, then check **Observations** > **Session At A Glance**

Find the peak of the pulsar using your cursor, then close the window and set the time of the peak to your start time. As for the day, pick a date about 3 days in the future. Observations must be submitted between 2 and 30 days before the scheduled time, so choose a few days in the future if you will be submitting your observations the same day as creating this file. After setting the start time, set the duration to 1 hour with ``01:00:00``. Reopen **Session At A Glance**, and the sinusoid should be replaced with an approximately horizontal line. Make slight alterations to the start time if the line has a significant slope. 

If you know your pulsar is exceptionally bright, the duration can be shortened to 30 minutes, ``00:30:00``. Likewise, if your pulsar is exceptionally dim, set the duration to 2 hours, ``02:00:00``. 	

RA: Retrieve the right ascension from the [Australia Telescope National Facility website](https://www.atnf.csiro.au/people/pulsar/psrcat/) (ATNF Pulsar Catalog). Scroll down to "Pulsar names" and enter your pulsar name, and click "Get Ephemeris." You may think you are getting results for another pulsar, but the catalog simply uses synonyms for the pulsar names given in different epochs. For example, searching for PSR B0655+64 renders results for PSR J0700+6418, but these are the same object. Fill in with the value for "RAJ," where the "J" indicated the J2000 epoch. 
	
DEC: Likewise, retrieve the "DECJ" value from ATNF, making sure to add the "+" or "-" sign. 

You will be creating TWO obsevation files, 1 for each tuning, with pairs (35.1, 49.8) MHz and (64.5, 79.2) MHz. All other information is the same, simply change the Session ID number n+1 for the second file. 

Tuning 1: ``35.1`` OR ``64.5`` 
Tuning 2: ``49.8`` OR ``79.2``

Navigate to **Observations** > **Advanced Settings**. Carefully scroll down to DRX-Specific Information. Change BEAM to ``1`` for your first file and to ``2`` for the second file. This predetermines the two beams used in by the LWA. When the beam has been chosen, click **Ok**. 

Now, you will check that all the information is correct. Navigate to **Observations** > **Validate All**. You should get a pop up: "Congratulations, you have a valid set of observations." Click **Ok** to close the window. If you don't get "Congratulations...", the error description will be displayed in the Terminal window. Make the needed changes until validation succeeds. 

Now to save the file, use **File** > **Save As**. Name the file ``<date>_<your name>_<pulsar name>_low.sdf`` OR ``<date>_<your_name>_<pulsar name>_high.sdf`` respectively for the two tunings. Be sure to add the ``.sdf``, as this is not automatically added as the file type when saved. Before saving, navigate to your directory (``/<computing ID>)in the file folder search bar. Click "Create Folder" and name it "Observations". Open "Observations" and now click **Save**.

Great! You can now check that the SDF (Session Definition File) is in 
```sh
/uva_students/<your computing ID>/Observations
```
Now, make the second observation file with all the same information except Session ID Number n+1, higher tunings, BEAM = 2, and file name `` *_high.sdf``. A shortcut to doing this is to open your `` *_low.sdf`` file, make the adjustments, and resave it as a new file. This prevents having to reenter the RA/DEC and start times. 

Once both files have been saved, navigate to **File** > **Quit** to close the SDF GUI. 

### Transferring Files

We will be submitting our SDF files to a LWA website that automatically schedules each valid observation file. However, to upload files from the LWA network, we must save the SDF files on the ``lwalab`` computer, rather than a ``ucf``. 

To transfer these files to the local system, exit the ``ucf`` via

```sh
$ logout
```

Your command line should now read

```sh
uvastudent@lwalab:~$
```

To transfer the files, we will use ``scp``, the secure copy command. To use ``scp``, input ``scp <original location of file(s)> <new location of file(s)>``:

```sh
$ scp uvastudent@lwaucf<#>:/home/uvastudent/uva_students/<your computing ID>/Observations/<date>_<your name>_<pulsar name>_* /home/uvastudent/observation_files/Fall2019
```

Hit return/enter, and you will be prompted for the password to the ``ucf``. Enter the password to allow the ``scp`` to go through. 

Inserting the wildcard operator `` * `` into the file address allows for both tuning observations to be copied at once. The file destination ("Fall2019") will vary from year to year. 

Success! You can now check that your file is in the right place on the local ``lwalab`` server. 

### Scheduling Observations

To schedule observations, we must upload the files through the remote server. Open the Browser within the X2GO LWA desktop. Go to ``http://fornax.phys.unm.edu/lwa/validator/index.html``

Click "Choose File". In the pop-up window, navigate to ``uvastudent>observation_files>Fall2019``
