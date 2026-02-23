# MEG : Chapter 4 - Preprocessing and epoching

### Contents

* [Contents](#Chapter4Preprocessingandepoching-Contents)
* [Know your experiment (triggers, trial length)](#Chapter4Preprocessingandepoching-Knowyourexperiment(triggers,triallength))
* [Reject external noise](#Chapter4Preprocessingandepoching-Rejectexternalnoise)
* [Interpolate missing channels (only for sensor analysis)](#Chapter4Preprocessingandepoching-Interpolatemissingchannels(onlyforsensoranalysis))
* [Downsample your data (save space and speed-up future analysis) - OPTIONAL](#Chapter4Preprocessingandepoching-Downsampleyourdata(savespaceandspeed-upfutureanalysis)-OPTIONAL)

### Know your experiment (triggers, trial length)

After having [detected all the artifacts time period and all the bad channels](Chapter-3---Cleaning-data_21139992.html) in your dataset, you can define your trials according to your experiment.

Here, the experiment is....

* [Demarchi Gianpaolo](https://im.sbg.ac.at/display/~b1019581) TODO: experiment description and triggers meaning?

Here is the code to read, cut and reject bad trials and channels:

### Reject external noise

```java
%% Apply signal space projection (SSP): reject external noise (if needed)
%-------------------------------------------------------------------------
cfg = [];
cfg.inputfile = fileName;
data_tr_cleaned_ssp  = obob_apply_ssp(cfg, data_tr_cleaned);
```

### Interpolate missing channels (only for sensor analysis)

After, having removed bad trials and channels we interpolate rejected channels using a home-made wrapper function to _**ft_channelrepair**_.The function basically automatically detects the labels of the removed channels and replaces it in the default version by the average of its neighbours (of the respective sensor type) taking into account the different sensor types. The "repaired" data will now include a 'label' field with all 306 channels and an 'origLabel' field that includes all channels without the discarded ones. This field will become important for source analysis later.

[* **obob_fixchannels**__takes the information from 'hdr' field, which may be not updated if you discarded channels in different moments of the analysis. If this is the case, you should use the original **ft_channelrepair**__(remember that it doesn't automatically store information about the discarded channels, so you might want to add it manually). cfg.method = 'distance' doesn't exist anymore]

### Downsample your data (save space and speed-up future analysis) - OPTIONAL

You can also decide to downsample your data prior to run future analysis.Here we can downsample to 512Hz.
