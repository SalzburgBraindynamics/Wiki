# MEG : Chapter 3 - Cleaning data

### **Contents**

* [Contents](#Chapter3Cleaningdata-Contents)
* [Artifact detection: Introduction](#Chapter3Cleaningdata-Artifactdetection:Introduction)
* [Read continuous data and apply SSP](#Chapter3Cleaningdata-ReadcontinuousdataandapplySSP)
* [1st data visualization](#Chapter3Cleaningdata-1stdatavisualization)
* [Cut your data into big chunks (just for the artifact detection)](#Chapter3Cleaningdata-Cutyourdataintobigchunks(justfortheartifactdetection))
* [Identify and remove bad channels](#Chapter3Cleaningdata-Identifyandremovebadchannels)
* [Automatic artifact detection](#Chapter3Cleaningdata-Automaticartifactdetection)
* [Compilation of your previous work](#Chapter3Cleaningdata-Compilationofyourpreviouswork)
* [2nd data visualization](#Chapter3Cleaningdata-2nddatavisualization)
* [Reject bad trials and channels by using this artifact detection procedure](#Chapter3Cleaningdata-Rejectbadtrialsandchannelsbyusingthisartifactdetectionprocedure)

### Artifact detection: Introduction

Of course you as an experimenter will try your very best in getting the cleanest data possible. Nevertheless some artifacts are inevitable and they come in different flavors, e.g.:

* Physiological artifacts such as blinks or ECG
* Environmental noise that will mostly affect your magnetometers
* Channel noise such as channel jumps (usually affecting a few trials) or "bad channels" (that are basically noisy over the entire experiment)

With regards to physiological artifacts some are easier to control (e.g. eye blinks) whereas others are inevitable (e.g. you can't ask the participant to stop the heart beating). While the most efficient way to remove occasional artifacts (such as blinks) is simply to remove those trials from the data, you can consider using e.g. ICA to remove other constant artifacts such as ECG. Whether you do so depends on to what extent you think that this factor is a problem in your analysis (e.g. do you think that there will be a systematic ECG difference between conditions / groups; in particular when such artifacts are time-locked to a specific event they are more problematic).

Our Neuromag system comes with different challenges with regards to this preprocessing step:

* Mixture of different sensor types
* Instability of external noise

With regards to the different sensor types you can of course choose to only read data from one sensor type in the first place (by setting cfg.channel to 'meggrad' or 'megmag' in ft_preprocessing), which is not uncommon (many papers only publish gradiometer data). But at the moment the MEG-group tries to keep the information and data together as long as possible. The instability issue poses particularly big problems when you are interested in sensor level analysis but solutions exists: such as the signal space projection (SSP) provided by Elekta system and implemented in **obob_apply_ssp**function.

While automatic thresholds or template matching procedures can be used to detect trials with certain artifacts, we think that the best way at the end of the day is still by visual inspection. Since this really is the only step in which you directly interact with the data, you should be careful during this step.

### Read continuous data and apply SSP

Now you should know how to read the whole raw data (from 1 block) so let's go:

If you have also recorded EOG, read and separate it here:

Now let's remove some external noise prior to perform artifact detection:

### 1st data visualization

We can have our first data visualization using ft_databrowser. The all point of this step is to appreciate the general quality of your data, so let's have a look to 30sec long data segments.Additionally, this first visualization can be used to identified the labels of clearly bad sensors

Do not delete the output of this function because you will use it later to concatenate all your artifact detection results:

You probably need to increase the vertical scale of the figure by clicking in the button "vertical" and enter [-1.5e-11 1.5e-11]Then, one thing you might is that one of the channel is quite different from the others and show a chaotic time course with high amplitude all along the data.This might be "bad channel" for this dataset that we should be removed later. Now you can access to this channel's name by clicking in right bottom button "identify" and clicking in the channel with the cursor.The only thing you know now: there is a potential bad channel for this dataset and its label is: MEG2542

![](attachments/21139992/21140018.png)

### Cut your data into big chunks (just for the artifact detection)

Now, it's better to artificially define "trials" or chunks in order to navigate efficiently with the interaction option of **ft_artifact_zvalue**.Just run the code below, note that you can adapt the number of chunks according to your preference, here we are using 20 chunks (~24sec each for this dataset)

Again, only if you have EOG, chunk it same as the data:

### Identify and remove bad channels

Even if you can clearly eye-balling bad channels with ft_databrowser, you might want to use quantified metrics in order to discriminate bad and good channels to remove from your data before going further.Such metrics can be found by using **obob_rejectvisual.**We will use this function in order to open an interactive window with summary metrics (i.e. variance, kurtosis, z-value...) computed for each channel and for each "trials" (here "trials" = chunks).In order to see how much some channels are different from the others according to these metrics. Don't forget that here we want to remove channels with constant "weird behavior" along the whole time-course of the data (across all chunks).You are looking for channels with constant high value along the dataset, you will take care of specific punctual jumps and artifacts later.In order to make all chunks' noise comparable you can remove some high noisy chunks in order to get rid of the between chunks variability (see NOTE statement below)Such a channel should have a data distribution with higher variance compared to the others and it could also present several jumps. Then, the two relevant metrics are: the variance and the kurtosis (measure of peakedness).**NOTE:**

* You can remove "trials" (data-chunk) during this procedure in order to make "trial-by-trial" noise variance as much homogeneous as you can and facilitate data observation to identify channels that are noisy all along the dataset (not only on a specific chunk).

Here is the code you can use:

This is what you should see (useful to have a large monitor):

![](attachments/21139992/21140026.png)

In the 'summary' view you can see three figures. The outer right displays the relevant metric (default is variance, but you can choose within the window) for each channel across the entire epoch.In the lower panel the same metric displays the metric for all trials (in our case we have 20 "trials" = our 20 data chunks).The upper left figure is then the channel x trials depiction in color coding.

Bad channels can be detected either in the upper right window or in the color coded window as horizontal lines with extremely 'hotter' colors.In this depiction you can clearly see the previous outlier channel presenting a clear higher variance compared to the others.You can also check for the kurtosis metric and once again the same channel it's clearly an outlier compared to the others.Basically you can remove outlying channels, simply by holding the mouse button and drawing a window over the relevant channels data points. So you can remove this outlier channel. Then you can track which channel you removed in the rejected channel section (right hand side), as expected it's the MEG2542.**NOTE**:

* The computation of the metric sometimes does take time. Don't be impatient and start clicking around wildly, just wait for the refresh of the figure plot.
* Channels removed will be displayed on the right hand side. You can toggle them in and out of the data by simply typing the according channel in the relevant field.
* When you remove a data point the all summary figure will re-scale automatically the axes.

This is what you should see after removing the bad channel:

![](attachments/21139992/21140029.png)

Sometime one can easily identify a 'dead' channel (not in this example), i.e. one channel sticking at 0 in the upper right image. If this the case, you can also remove it accordingly as you did before.

When you are done click "Quit" button on the left bottom side (do not just close the window). You can now check **data_ssp_chunks_okchan** (the output of **obob_rejectvisual** function) and you will see that the channel has been indeed removed. Importantly the 'label' field now only has 305 instead of 306 cells.Also you can keep track of the good channels labels and indices for later (by using a structure named: **okchan**).

### Automatic artifact detection

Now we can look for well-known artifacts automatically by using fieldtrip z-value artifacts detection function ([see here for more details](http://www.fieldtriptoolbox.org/tutorial/automatic_artifact_rejection)).Here we fixed the initial z-value threshold cutoff at 15 but you should also use the interative mode in order to adapt this threshold to your data.The different type of automatic artifact detection methods you want to use on your data is up to you!For now, you can just run the script below which performs automatic detection for occasional channel jumps, muscular and blinks artifacts sequentially.

First, the occasional channels jumps is performed on Magnetometers and Gradiometers separately.After running the code below a window will open and allow you to adapt the threshold and control for the artifact detection (by rejecting or by accepting the automatic detection results)

This is what you should see:

![](attachments/21139992/21140156.png)

This is the interactive figure of **ft_artifact_zvalue**. The left panel shows the z-score of the processed data, along with the threshold. Suprathreshold data points are marked in red. The lower right panel shows the z-score of the processed data for a particular "trial" (chunk in our case), and the upper right panel shows the unprocessed data of the channel that contributed most to the (cumulated) z-score. You can browse through the trials (chunks) using the buttons at the bottom of the figure. Also, you can adjust the threshold, and manually keep/reject trials.

**NOTE:**

* In our case we don't want to use "reject full" button (this will affect the whole chunk), so we reject only part of the data by using "reject part" button
* Adapt the threshold value in order to include the main peaks on the left panels (click on "threshold" button to change the threshold value)
* Take your time to examine each supra-threshold data and decide wether you keep it as a jump artifact or if you can keep this portion of signal by clicking in "keep trial" button

In this example data set, some data points are detected to be a 'jump' artifact, although most of them are not jump artifact, try to reject only occasional jump artifacts. A typical jump artifact can be seen below:

![](attachments/21139992/21140162.png)

Explore and adapt your artifact detection for both sensors type, then you can switch to another type of artifact detection:

The muscular artifact detection can be performed on Magnetometers only.Like you did before, adapt the rejection threshold and exclude only "muscular artifact".

Finally, we can try to detect blinks time period by using an EOG channel (if you have one) or Magnetometers:

If you have EOG, first append it to the data:

If you do not have EOG, skip the above step and instead use magnetometers to detect blinks:

(Important: you have to use the code below whether you have EOG or not. If you do have EOG, edit the 3rd line as indicated. If not, keep it as it is)

### Compilation of your previous work

During this step, you put together all the outputs of the automatic artifact detection and you will control visually the quality of this detection.**NOTE:** here we are using the previous cfg_art structure which keep track of all our artifact detection

### 2nd data visualization

Now you can have a second look at your data with the results of your semi-automatic artifact detection.You can use this second visualization to remove or add artifact time period by exploring all data chunks.**NOTE**: here I think it's better to look at different type of sensor separately by using the vertical viewmode but it's up to your taste!

This is what you should see (useful to have a large monitor):![](attachments/21139992/21140166.png)

Using this figure, you can increase the vertical scale (e.g. to [-1.5e-13 1.5e-13]) and you can navigate through all the previously identified artifacts by clicking on the color boxes (top right side) specific to each artifact detection method.You can remove or add artifact periods by selecting a time period and by clicking inside your selection.

When you are happy with your artifact definition you can close the window and saved your work:

### Reject bad trials and channels by using this artifact detection procedure

Now you can basically load and read the corresponding dataset, define your real trial definition and reject bad channels and artifacted trials based on this artifact detection procedure we have done!This is how you do it:
