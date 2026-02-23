# MEG : Chapter 2 - Reading data

### **Contents**

* [Contents](#Chapter2Readingdata-Contents)
* [So, let's get started...](#Chapter2Readingdata-So,let'sgetstarted...)
* [Functions and structures](#Chapter2Readingdata-Functionsandstructures)
* [Let's read some data!](#Chapter2Readingdata-Let'sreadsomedata!)
* [Let's take a look at the data structure](#Chapter2Readingdata-Let'stakealookatthedatastructure)
* [Known bug](#Chapter2Readingdata-Knownbug)

### So, let's get started...

Prepare a location for your scripts and functions and for the data you are going to produce

The first thing you want to do is to look for a nice place to put the scripts and functions you are about to write and a separate location for the data files you are about to produce. You should really choose two different locations for these as (in most of the cases) you want to keep scripts, functions and raw data on a backed up storage while the data you are about to produce can be reproduced easily so you can safely put it somewhere-else.... Create those two folders. In the folders for your scripts, create another folder called _functions_ and then you are set.

### Functions and structures

Almost all Fieldtrip functions follow the following convention that a structure containing configuration parameters is passed along either alone or with data to which the parameters are to be applied. I.e. you will almost always see something like this:

OR

OR

### Let's read some data!

In this section, we will start with the basic commands to read and explore data.To read and filter all the continuous data, all we have to do is this:

This might take a minute to run and you will see some output on the screen. There might be some warnings that you can safely ignore. As long as there are no errors, everything is going to be fine. We called two different functions: _ft_definetrial_ and _ft_preprocessing_. In short:

* the first one determines what data to use, whether to read in trials or the whole data → here we are reading the whole raw data.
* the second one does the actual import of the data: this is also a good place to define some filtering parameters (high and low pass filter, if relevant) → here we are filtering the data with highpass 1Hz

So, what is the result of all this? The last line of code you see up there has finally read in all the data from the file and put it in the variable _data_. We will take a closer look at what this structure contains. First, let us visualize the data we just read. FieldTrip provides a nice function called _ft_databrowser_ for this.

### Let's take a look at the data structure

Ok, so we managed to read in the data from the file and it is apparently stored in the structure called _data_. Now we can look, discover and explore every aspect of the data. So, let's have a look what _data_ looks like. Just enter _data_ at the Matlab prompt. The command window now should look like this:

You can see that some of the fields are simple variables that contain numbers (fsample) or vectors (sampleinfo). Others contain cells (label) and others contain structures.If you want to explore one of the fields, just enter its name after _data._ and make sure, not to forget the "." to separate the structure's name from the field name. So, let's go through the fields and see what they represent:

1. data.hdr: This is additional information about the data that comes from the file. Normally you do not have to bother.
1. data.label: These are the labels of the channels in the data.
1. data.time: The time in seconds of every sample.
1. data.trial: The actual data. This fields holds all the trials of your data. At the moment, we only have one trial which is your complete data [channels x time].
1. data.fsample: The sampling rate.
1. data.sampleinfo: The first and last sample indices of your trial definition (in our case: the first and the last one of the raw data)
1. data.grad: This holds information about the gradiometers. E.g., the position and orientation of each of the is stored here.
1. data.cfg: This holds the cfg structure that was used in the function that created the structure.

Take your time and play around with this data structure. Discover the information it has. As an exercise, try to plot a few samples of one channel just using the _plot_ command. Try adding the right time axis....

### Known bug

If you work with MATLAB on your local machine using windows it can happen that you are not able to read .fif files stored on the server (mounting via [sftpNetDrive](Best-practices-for-playing-with-MEG-data_23859778.html))! [See here the error message.](attachments/21139942/27408084.png) There is currently no easy solution that we know of. In some cases you may rely on using the bomber what can be time consuming in case you rely on using the graphical user interface / GUI) and you might even encounter problems with the GUI such that figures/ data are not shown appropriately.

Some workarounds:

For doing artefact cleaning via your local MATLAB in an effcient way, simply store the data / fieldtrip structure as a mat file after you did the preprocessing (see above: data = ft_preprocessing(cfg))! The mat file you can easily load from the server.

For doing the [coregistration](Source-analysis_21140202.html) you can extract and save the relevant information (e.g. headshape) stored in the fif file. This you have to do on the bomber. Then you can continue working via your local MATLAB! Note that this works so far only when you have a MRI of your subject. When you do not have an MRI the obob_coregister functions asks you per default to provide the full fif file path for the cfg.headshape. Of course, this won't work due to the bug ([see error message](attachments/21139942/27408084.png)). The good thing in that case is that you can simply do the coregistration on the bomber as the GUI for that purpose works fine enough.
