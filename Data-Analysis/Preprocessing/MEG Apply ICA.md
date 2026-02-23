# MEG : Chapter 4b - Applying independent component analysis (ICA)

* [Applying ICA on MEG data recorded via Neuromag](#Chapter4bApplyingindependentcomponentanalysis(ICA)-ApplyingICAonMEGdatarecordedviaNeuromag)
* [Here are some points to consider before applying ICA:](#Chapter4bApplyingindependentcomponentanalysis(ICA)-HerearesomepointstoconsiderbeforeapplyingICA:)
* [The general coarse strategy when applying ICA during preprocessing:](#Chapter4bApplyingindependentcomponentanalysis(ICA)-ThegeneralcoarsestrategywhenapplyingICAduringpreprocessing:)
* [ICA methods and which one to choose and why?](#Chapter4bApplyingindependentcomponentanalysis(ICA)-ICAmethodsandwhichonetochooseandwhy?)
* [How to apply (the infomax) ICA via obob_ownft?](#Chapter4bApplyingindependentcomponentanalysis(ICA)-Howtoapply(theinfomax)ICAviaobob_ownft?)
* [Further reading](#Chapter4bApplyingindependentcomponentanalysis(ICA)-Furtherreading)

**Note**: To avoid unforeseeable issues with future fieldtrip releases, it might be best to convert the units of your grad field (which presumably should be in 'cm' after preprocessing) into 'm' before calculating ft_componentanalysis.

### **Applying ICA on MEG data recorded via Neuromag**

Some backround info and theory behind the ICA you will find [here](http://arnauddelorme.com/ica_for_dummies/) and [here](https://sccn.ucsd.edu/~jung/Site/EEG_artifact_removal.html).

This tutorial is based on what has been shared via the [CIMeC](https://wiki.cimec.unitn.it/tiki-index.php?page=ICA%2BData%2BCleaning) wiki by the OBOBs some years ago and has been updated to a more or less degree by a MEG user who has never applied ICA before. Sources for everything what you can read here are provided in the text. So feel free to update and edit based on your expert knowledge and/or helpful sources.Optionally to the standard visual artefact rejection, MEG data can also be cleaned using ICA by detecting components that cleanly capture e.g. ocular or cardiac activity. This should, however, only be applied when deemed absolutely necessary (e.g. very long ISIs, ECG removal from resting state data etc.) since ICA changes the measured data.

### **Here are some points to consider before applying ICA:**

* **Think carefully about what data to feed in the ICA**(in this chapter referred to as data_for_ica).There are**two aproaches**how to apply ICA**when having a sensor mix** (gradiometers and magnetometers → because of the different magnitudes). a) One approach is to estimate the independent components separately for gradiometers and magnetometers as it was done [here](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4972935/). b) Another approach is to estimate the independent components on all the sensor types at once (which is advantageous concerning the time to invest on screening and choosing the independent components to reject from your data as you do not have to do it for all the different sensor types separately!)in that case you should make sure to bring the different magnitudes on the same scale (see e.g. [here](https://wiki.cimec.unitn.it/tiki-index.php?page=ICA%2BData%2BCleaning); cf. a discussion on that [here](https://mailman.science.ru.nl/pipermail/fieldtrip/2010-March/002701.html))however, you do not necessarily have to do this manually. in fact, some algorithms take care of that issue (cf. the discussion on that [here](https://mailman.science.ru.nl/pipermail/fieldtrip/2012-April/005016.html)): in particuar: the [runica implementation](https://sccn.ucsd.edu/~jung/tutorial/runica.htm) (i.e. the EEGlab function for the ica method _infomax_used in fieldtrip) performs a [whitening of the data](http://arnauddelorme.com/ica_for_dummies/) prior to the decomposition → this addresses the difference in magnitude issue**Provide enough data to yield stable components**It might not really matter if you feed in epoched or continous data but make sure to feed in enough data! As a general rule,**finding N stable components (from N-channel data) may require more than 3*N^2** (according to Tony Bell, inventor of ICA Infomax) data sample points (at each channel). Most scientists who fail to find reliable components using ICA do not follow this rule (cf. [here](http://cognitrn.psych.indiana.edu/busey/temp/eeglabtutorial4.301/maintut/ICA_decomposition.html)). However, when having >300 MEG channels and when downsampling the data (e.g. to 250 Hz) to speed up processing one can easily fall below this critical threshold! Anyway, if you want to have the best ICA output possible, make sure to stick to the **3*N^2** rule (e.g. avoid downsampling). From my experience it is worth it though you will end up with lots of data to (permanently) store and it needs much more computation time but I discovered more independent components to reject from my data (though in that case I additionally added two more options, which might have contributed to the better outcome - _cfg.runica.extended = 1_ &  _cfg.runica.stop = 1E-7_; see below for details)!**Think about to which extent to filter your data** (differently compared to your previous preprocessing steps). Note that there seems to be no consensus what filter to apply. Here some directions:a) Components seem to be better identified when applying a 1-Hz highpass filter. The "[mne wiki](https://martinos.org/mne/stable/generated/mne.preprocessing.ICA.html)" says "ICA is sensitive to low-frequency drifts and therefore requires the data to be high-pass filtered prior to fitting. Typically, a cutoff frequency of 1 Hz is recommended." For a further discussion on that see e.g. [here](https://sccn.ucsd.edu/pipermail/eeglablist/2011/004429.html). In case you are not interested in filtering your (preprocessed) data with such 1-Hz hp filter, you can apply the filtering only for the ica-preprocessing step and remove the identified components from your less agressively filtered (preprocessed) data (in this chapter referred to as → data_preproc).  Be also aware of what filtering does to your data and think carefully about whether you apply the 1-Hz hp filter to epoched data (as filtering may distort your data especially at the edges of the epochs; cf. [here](https://im.sbg.ac.at/display/MEG/Chapter+4a+Preprocessing)) or to continous datab) However, some reasearchers apparently feed in data filtered with a 0.1-Hz hp-filter e.g. as it was done in the [ft-tutorial here](http://www.fieldtriptoolbox.org/tutorial/salzburg?s[]=ica)**When applying ICA on maxfiltered data, you may need to reduce the dimensionality of the data** as otherwise the results of the ICA decomposition might contain complex values ([see here for details](http://www.fieldtriptoolbox.org/faq/why_does_my_ica_output_contain_complex_numbers?s[]=complex&s[]=numbers))
* **When using ICA for source analysis** a) it is advantageous for any source localization that the **same components** are **removed from the leadfields** (cf. [here](http://www.fieldtriptoolbox.org/project/guidelines/paper/preprocessing?s[]=ica#caveats)) For ICA the new leadfield is: Lnew=A*(EYE- CR)*pinv(A)*Lold. Here, Lnew is the new leadfield, corrected for ICA artefacts, Lold is the old leadfield. A is the full estimated ICA mixing matrix (including whitening etc.), pinv(A) is the pseudo inverse of A, EYE is the identity matrix and CR is a matrix that has zeroes everywhere except on the diagonal entries that correspond to ICA components that have been removed from the data. If a dimension reduction via PCA is done before ICA the formula should be: Lnew= pinv(Q*(EYE- CP)) * A * (EYE- CR) * pinv(pinv(Q*(EYE-CP))*A) * Lold where Q is the PCA whitening matrix and A is the ICA mixing matrix obtained on the dimension reduced data. CP is a matrix with zeros everywhere except for those diagonal entries that correspond to removed PCA components.**How to do that in practice?**You can use the .grad field from the output of ft_rejectcomponent for the computation of the leadfield. The balancing matrix grad.tra will contain the information that will be used for the correction of the leadfields. (see[here](https://mailman.science.ru.nl/pipermail/fieldtrip/2012-February/004833.html))b) beamformer sourceanalysis requires a **regularization factor** (cfg.xxx.lambda). But see [here](https://mailman.science.ru.nl/pipermail/fieldtrip/2018-August/012319.html) a recent note that it might also fail....
* The **same ICA components should be used for all conditions** within one subject. However, when you deal with MEG data, you need to consider that across blocks the head might be in different positions (depending on the subject's head movement). Thus, the stationary MEG channels could capture activity from different head positions across blocks. So, you probably want to **apply the ica blockwise** rather than on the whole data set of one particular subject. So, having a study in which all conditions are included within one block might be the perfect situation.
* It is more elegant to **apply ICA to either all or none of the subjects**. Don't mix. When you have one or two "bad" subjects (e.g. plenty of artefacts), then remeasure another two.
* There exist **different ica methods** (for details see below) and you might want to think and inform yourself about which one is best for your specific case

### **The general coarse strategy when applying ICA during preprocessing:**

* Choose data upon which you want to calculate the ICA components (see above) → data_for_ica
* Clean the data_for_ica, discarding: a) in particular very extreme artefacts of NONPHYSIOLOGICAL origin. This will mainly be channel jumps, that will be recognized easily once you encounter one. b) dead or extremely noisy channels. Such data will cause the ICA to not converge and lead to nonsense results.c) extreme, non-stereotype physiological artifacts (e.g. stemming from muscle activity). Note that you should NOT discard artifacts that you want to remove via ICA, e.g. heart and eye activity, as otherwise the ICA might have troubles finding clear artifact "examples" (see [here](https://sccn.ucsd.edu/pipermail/eeglablist/2015/009720.html)).
* Run ft_componentanalysis on data_for_ica.
* Identify relevant artefact components mainly by using topographic and time-course information via ft_databrowser.
* Reject artefact components from the preprocessed data using ft_rejectcomponents (see above).

Now you are back in the tutorial pipeline and can continue with your analyses.

### **ICA methods and which one to choose and why**?

There exist different methods (e.g. infomax, jader, africa, etc). Fieldtrip seems to apply per default the ica method [infomax via the function runica](https://bitbucket.org/sccn_eeglab/eeglab/src/a2983535293a3bd3605e96a806182d6d6a4a2594/functions/sigprocfunc/runica.m?at=master&fileviewer=file-view-default) implemented in the [EEGLAB toolbox](https://sccn.ucsd.edu/wiki/Chapter_09:_Decomposing_Data_Using_ICA). Based on what is written on the [eeglab webpage](https://sccn.ucsd.edu/wiki/Chapter_09:_Decomposing_Data_Using_ICA), infomax seems to be a good choice, though there might be more optimal / appropriate ones available nowadays given that the infomax was invented in 1995 and there might have been quite some progress in improving it. E.g. there are probably some more automatic methods that might come with the advantage that the identification of the artifact component is not (entirely) a subjective judgement.  However, in case one is interested to mainly remove sources  that are easy to identify via time course and topoplots such as eye blinks, eye movements, etc, the infomax method seems to be still a good choice.

### **How to apply (the infomax) ICA via obob_ownft?**

This subsection assumes that you have read and understood all the details above. So, you need some knowledge what data to actually feed in and what (additional) preprocessing steps might be necessary!

Here, we feed in continuous sensor data from one 5-min (experimental) block. Data were filtered (100 Hz lp filter, 1 Hz hp filter), downsampled (250 Hz; though this might be less optimal as the 3*n_channels^2 [see above] was not met) and include magnetometers and gradiometers). Data were cleaned from bad/dead channels and from extreme artifacts. The data still contain train noise, line noise, and (non-extreme) physiological artifacts such as activity stemming from the heart, eyes and, muscles → data_for_ica.

Note, if you did apply a 1-Hz highpass filter on the data_for_ica, etc. only for the purpose of applying ica, while later on you want to reject the components from your preprocessed (less agressively filtered and cleaned) data → data_preproc, there is no need to save the data_for_ica. That is, the preprocessing for the data_for_ica you can do in the same script as what follows next:

1) In the first step, we want to**decompose the data into independent components**.

```java
cfg = [];
cfg.method = 'runica'; 
% cfg.runica.extended = 1; 
% cfg.runica.stop = 1E-7

ica_components = ft_componentanalysis(cfg, data_for_ica);
```

Note that this step can take a few minutes (when running on the bomber). In the current case it took > 7 min! The output "comp" structure resembles the input data_for_ica structure, i.e. it contains a time course for each component and each trial. Furthermore, it contains the spatial mixing matrix. I.e. the data can also be more or less huge in size. Don't forget to save the ica_components! This code you can run on the cluster to be more efficient!

Note that the "runica" Infomax algorithm can only detect supergaussian (i.e. peaky) distribution of activity. If there is strong line noise in the data, the EEGlab people reccomend to use the option **cfg.runica.extended = 1**, so the algorithm can also detect subgaussian sources of activity, such as line current and/or slow activity. Another option the EEGlab people often use is the stop option: try **cfg.runica.stop = 1E-7** to lower the criterion for stopping learning, thereby lengthening ICA training but possibly returning cleaner decompositions, particularly of high-density array data. The EEGlab people also recommend the use of collections of short epochs that have been carefully pruned of noisy epochs (cf.[here](http://cognitrn.psych.indiana.edu/busey/temp/eeglabtutorial4.301/maintut/ICA_decomposition.html)).

**Important**: if you were to run the ica a second time, ICA decompositions under _runica_ would slightly differ. That is, the ordering, scalp topography and activity time courses of best-matching components may appear slightly different. This is because ICA decomposition starts with a random weight matrix (and randomly shuffles the data order in each training step), so the convergence is slightly different every time (cf.[here](http://cognitrn.psych.indiana.edu/busey/temp/eeglabtutorial4.301/maintut/ICA_decomposition.html)).

**Is it beneficial to apply PCA before ICA?**In fact, several researchers / labs seem to do this (for several reasons: e.g. [after maxfiltering](https://mailman.science.ru.nl/pipermail/fieldtrip/2013-March/006268.html) because data may be rank deficient; or because one gets fewer components to inspect / "it is easier to interpret" [a la Anne Keitel]), etc). Running PCA before ICA is even implemented in****the function****_ft_componentanalysis_ via cfg.runica.pca (to be specified as usually less than 50 (maxfiltered data rank ranges from 59 to 79!) when confronted with maxfiltered data!). However, think critically before you do this. The 2018-discussion from mne-phyton developers [here](https://github.com/mne-tools/mne-python/issues/4856) and the 2018-paper from big shots in the EEG domain [here](https://www.sciencedirect.com/science/article/pii/S1053811918302143?via%3Dihub) might give you a starting point for this.

2) In the second step, we want to **inspect and identify the independent components** / sources (e.g. eye, heart, etc) based on their topography and time course.

```java
cfg = [];
cfg.blocksize = 10; % 10 seconds of time course data segments to look at
cfg.layout = 'neuromag306mag.lay'; % use the layout for magnetometers for plotting
cfg.viewmode = 'component';
% cfg.zlim = 'maxabs';

ft_databrowser(cfg, ica_components)
```

This code you can apply on the bomber. You can browse through the independent components and the data segments.

![](attachments/27411115/27411316.png)

You do not necessarily need to scroll through all the independent components (what can be time intense given that the number of channels = the number of components). Usually, it is enough to scroll through the first ~50 components (but test yourself and find the best routine). This is because the component order returned by _runica_ is in decreasing order of the MEG variance accounted for by each component. In other words, the lower the order of a component, the more data (neural and/or artifactual) it accounts for (for further reading, go [here](http://cognitrn.psych.indiana.edu/busey/temp/eeglabtutorial4.301/maintut/ICA_decomposition.html)).

As you will see in the figure below, some of the artifacts show a very characteristic time course and / or topography. Note that for the time course information, e.g. the heart is present in all the segments while for the eye artifacts and the train noise you might need to scroll through the segments to find them as participants (luckily) do not blink continuously and a train is also not passing by all the time the MEG lab.

![](attachments/27411115/27411317.png)

Note that if a component looks like a mixture of neural activity and artifacts, it is recommended to leave it in the data!

**Important:** when I run the ICA again meeting all the optimal criteria (i.e. **meeting the 3+n_channels^2 rule** via **NO downsampling** and adding the options **cfg.runica.extended = 1**& **cfg.runica.stop = 1E-7** I found more independent components. So, to achieve the best outcome possible you may want to stick to the recommendations by the infomax inventor (see e.g. [EEGLAB wiki](https://sccn.ucsd.edu/wiki/Chapter_09:_Decomposing_Data_Using_ICA)).

Make sure to manually write down and save all the independent components you want to remove from your data, e.g. in that form: reject_components = [3 6 10 13]!

**Some practical hint**: The scaling is the same for all components. As some artifacts like that of the train are super huge they kind of make the topoplots of other components less meaningful (see figure above and below). So, when you identified the component as e.g. a train (as it is the case here), write its value (here 1) down (to reject it later from the data to be analysed) and remove it from the Dataviewer: I.e. press "component" in the databrowser (bottom ~left). Then the following window appears:

![](attachments/27411115/27411502.png)

Mark the component (here runica001) and press "remove".

![](attachments/27411115/27411503.png)

Now the topoplot of other components become more meaningful. You can see here e.g. the typical topography for the heart beat artifact (bottom row).

3) In the third step we want to **reject the artifacts from the data**

```java
reject_components = [3 6 10 13]

cfg = [];
cfg.component = reject_components; 
data_ica_cleaned = ft_rejectcomponent(cfg, ica_components, data_preproc);
```

One routine that works is the following:

1. Load continuous, filtered (and downsampled) data
1. Select only MEG channels (EOG or ECG channel)
1. Remove ICA components
1. Continue with the preprocessing pipeline as if you were not applying ICA (i.e. epoch data, clean data from bad channels and (non-stereotype) artifacts (such as channel jumps, etc that you discovered in a previous preprocessing step via ft_databrowser and of which you stored the info where it occurs in your data), append epochs of different blocks, etc.
1. Store the data_ica_cleaned to work with them further.

Make sure to compare the outcome from the ICA cleaning pipeline (see left figure below) with those for which no ICA was applied (see right figure below) via e.g. ft_databrowser to see that removing the artifacts was successful. In the example below you see successful eye blink rejection. In case your data also included train artifacts, you may want to check successful rejection of those ica components as well!

![](attachments/27411115/32867062.png)![](attachments/27411115/32867061.png)

Now, we can simply use data_ica_cleaned in sensor space to calculate ERFs or TF representations.

When interested in beaming the data_ica_cleaned into source space, we need to make sure to not run into [caveats](http://www.fieldtriptoolbox.org/project/guidelines/paper/preprocessing?s[]=ica#caveats).

**Important:**Note that despite the big data size of ica_components it is necessary to store them permanently as you cannot rebuild it via your scripts (because ICA decomposition starts with a random weight matrix and randomly shuffles the data order in each training step)!

### **Further reading**

* [ICA to remove ECG artifacts](http://www.fieldtriptoolbox.org/example/use_independent_component_analysis_ica_to_remove_ecg_artifacts)
* [ICA to remove EOG artifacts](http://www.fieldtriptoolbox.org/example/use_independent_component_analysis_ica_to_remove_eog_artifacts)
* [Recommendations and caveats for preprocessing in general and in particular for ICA](http://www.fieldtriptoolbox.org/project/guidelines/paper/preprocessing?s[]=ica)
* [Tutorial on ICA via EEGLAB wiki](https://sccn.ucsd.edu/wiki/Chapter_09:_Decomposing_Data_Using_ICA)
* [ICA in source space](https://www.sciencedirect.com/science/article/pii/S1053811914006351?via%3Dihub): This has not been dealt with in the current chapter but is interesting to know and sounds in fact very promising:"source-space ICA is superior to sensor-space ICA and beamforming" (Jonmohamadi et al., 2014)"Source-space ICA is superior to source-space power mapping via beamforming for identification of multiple weak and strong sources. This is because source-space ICA can separate weak and strong sources and provide separate tomographic maps for each of the identified components, whereas the power map of the source-space via beamforming is unable to distinguish weaker sources in the presence of stronger sources. Source-space ICA is also able to localize sources with higher spatial accuracy than ICA plus dipole fitting/minimum-norm filter." (Jonmohamadi et al., 2014)
