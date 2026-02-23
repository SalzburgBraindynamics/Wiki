# MEG : Chapter 6 - Source analysis

This site is dedicated to doing simple source analysis without going into technical details. If you are interested in how this works, please go to the old site "[Source analysis](Source-analysis_21140202.html)".

Here, we just want to provide **a clear pipeline** that will get you from sensor data to source data.

### **What do you need to perform source analysis in the first place?**

1. MEG sensor data.
1. An anatomical structure of the brain (MRI).

### **Co-registration and headmodel**

First, you need to co-register the MEG data with the MRI data and create a headmodel. This is explained very well in our old "[Source analysis](Source-analysis_21140202.html)" page. Please follow the instructions there and then come back to this site.

### **Computing spatial filters**

This section is dedicated to the following two things that need to be done for getting source estimates of your data:

1. Calculating the source model
1. Calculating the leadfield
1. Using both to get source estimates of your data

If you want to know what exactly is happening here, I recommend the following video which really nicely explains everything:

The most important thing that changed to the old pipeline is the fact that the leadfield has to be calculated on the **RAW data**. This leads me to an advice to do the spatial filter calculation: **Calculate your spatial filters in a separate function and afterwards apply them to your data!**

I will now explain step by step how this function should be organized:

**1)** Load your fif-File from the Storage and apply the "preprocessing-function" from Fieldtrip with no specifications.

```java
cfg = [];
cfg.trialdef.triallength = inf;
cfg.dataset = fileName;
cfg = ft_definetrial(cfg);
  
data_raw = ft_preprocessing(cfg);
```

**NOTE:** If you have multiple trials, you should append them before going on, like this:

```java
cfg = [];
cfg.appenddim = 'rpt';
data_leadfield = ft_appenddata(cfg, data_raw{:});
data_leadfield.grad = data_raw.grad;
```

**2)** If you did an ICA or a visual inspection to identify and remove artifacts, now is the time to remove them. Here I will put in an example of how to remove ICA components.

```java
cfg = [];
cfg.component = rejected_components;
data_ica_rm = ft_rejectcomponent(cfg, data_ica.ica_components, data_raw);
```

**3)** Now you are doing the exact same thing as you are doing in the pipeline for you actual data:

* Use your trial-function to define your trials
* Change the sampling points to control for the delay between the trigger and the actual signal that is delivered to the MEG (auditory = 16.5 ms, visual = 9 ms).
* Resample the data.
* Append the data. This results now in preprocessed, resampled, epoched data. **NOTE:** This data is different from the data that we appended under 1)!

**4)** We now have 2 different datasets:

* data_leadfield: This is data that is rarely processed. This is needed to accurately produce source estimates and is used for calculation the leadfield.
* data_epoched: As already mentioned, this is preprocessed, resampled, epoched data that we are using for calculating the covariance matrices that are then passed on to the function "obob_svs_compute_spat_filters".

**5)** Now it is t ime to load the coregistration-files, the headmodel and also the template grid (I use the one with 2982 points, this is individual):

```java
load(/mnt/obob/staff/YOURNAME/YOUR_PROJECT/coregistration/coregister_SUBJECT_ID.mat', 'mri_aligned', 'hdm');

load mni_grid_1_cm_2982pnts.mat
template_grid = ft_convert_units(template_grid, 'm');
```

**6)** Now we need to use the function "ft_timelockanalysis" to compute covariance matrices that are then used later for the spatial filter calculation.

```java
cfg = [];
cfg.covariance='yes';
cfg.covariancewindow = 'all';
cfg.keeptrials = 'no';

data_filter = ft_timelockanalysis(cfg, data_epoched);

% always look after grad definition
data_filter.grad = data_epoched.grad;
```

**7)** Then we are calculating the sourcemodel based on the co-registration ("mri_aligned"):

```java
cfg=[];
cfg.mri=mri_aligned;
cfg.grid.warpmni='yes';
cfg.grid.template = template_grid;
cfg.grid.nonlinear='no';
cfg.grid.unit='m';
individual_grid = ft_prepare_sourcemodel(cfg);
```

**8)** Now we are calculating the leadfield. **Here it is VERY important that you do NOT take "data_epoched", but "data_leadfield":**

```java
cfg = [];
cfg.grid = individual_grid;
cfg.vol = hdm; % subject specific headmodel /volume
cfg.normalize = 'yes';

subject_leadfield = ft_prepare_leadfield(cfg, data_leadfield);
```

**9)** The last step is now the calculation of the actual spatial filter, using the leadfield and the source model:

```java
cfg = [];
cfg.grid = subject_leadfield;
cfg.fixedori = 'yes';
cfg.regfac = '5%';
spatial_filter = obob_svs_compute_spat_filters(cfg, data_filter);
```

NOTE: cfg.regfac introduces a regularization factor for "smoothing" the effects. The standard is 5%, but if you feel like your effects are strange or slightly missplaced, you can go up to 20%.

**10)** Then you just save the variables "spatial_filter" and "subject_leadfield" and you are done!

### **Applying spatial filters to your data**

The next step now is to apply your spatial filters to your data.

**1)** Load your **preprocessed data** - Not the one from the spatial filter calculation, because we did not save them, but the preprocessed data that you calculated in another function and saved as a .mat-file.

**2)** Load your **spatial_filters** that we just calculated.

**3)** Load the **template grid** that you also used in the spatial filter calculation (in this example the grid with 2982 points).

**4)** Then we are already at the last step - we are **applying our filters to our preprocessed data** (here it is called "meg"):

```java
cfg = [];
cfg.spatial_filter = spatial_filter;
datasource = obob_svs_beamtrials_lcmv(cfg, meg);
```

**5)** From this, you can go on just as in sensor space, e.g. doing time-frequency analysis or time-lock analysis.

**6)** Save this data as you would save it in sensor space.

### **Plotting your source data**

Let's assume now that we just did a simple timelock analysis trying to have a look at a visual ERF. Here is how you will get to your data plotted in source **for a grand average over all subjects**:

**1)** Load the data that we saved as "datasource".

**2)** Load your template grid, the standard-MRI and the atlas:

```java
load mni_grid_1_cm_2982pnts
template_grid = ft_convert_units(template_grid,'m');

load standard_mri_better
ft_convert_units(mri,'m');

atlas= ft_read_atlas('/mnt/obob/obob_ownft/external/fieldtrip/template/atlas/aal/ROI_MNI_V4.nii');
```

**3)** Insert the grid point positions, etc. into the individual subjects data:

```java
for i = 1:numel(subject_id)
  datasource{i}.pos = template_grid.pos;
  datasource{i}.inside = template_grid.inside;
  datasource{i}.dimord =  'chan_freq';
  datasource{i}.dim = template_grid.dim;
end 
```

**4)** Calculate the grand average using ft_sourcegrandaverage. In most of the cases, the parameter is either "powspctrm" or "avg". If you have another case, check with the senior scientists how to go on as in some

```java
cfg = [];
cfg.parameter = 'powspctrm';
grandavg = ft_sourcegrandaverage(cfg, datasource{:});
```

**5)** Now we are using again our own function "obob_svs_virtualsens2source" to beam the data into source. Here you have to chose your**frequency of interest or your time window of interest** (dependent on the data you have):

```java
cfg = [];
cfg.parameter = 'powspctrm'; %or 'avg'
cfg.atlas = atlas;
cfg.frequency = [ xxxx ];
cfg.latency = [ xxxxx ];
cfg.sourcegrid = template_grid;
cfg.mri = mri;
source_final = obob_svs_virtualsens2source(cfg, grandavg);

source_final.coordsys = 'mni';
```

**6)** In a last step, we then plot our source data onto an orthoplot:

```java
cfg = [];
cfg.funparameter = 'powspctrm'; %or 'avg'
cfg.method = 'ortho';
cfg.funcolormap = 'jet';
cfg.atlas = atlas;
cfg.crosshair = 'yes';

ft_sourceplot(cfg, source_final)
```

**7)** If everything runs smooth, you will likely get a result like this:

![](attachments/242270697/242270773.png)

**8)** This was it! If you need any further information or run into problems, just step by any senior scientist's office.
