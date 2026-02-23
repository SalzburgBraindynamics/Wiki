# MEG : 3b. Compute forward encoding model (TRF) - volume source space

Now we can estimate temporal response functions (TRFs) to predict our MEG response to an acoustic envelope. This also gives us a measure of neural tracking (prediction accuracy), which is the correlation between the predict and original MEG signal in every source.First, let's do some imports and load the data. We have an MEG dataset (lowpass 10Hz, highpass 1Hz, 100Hz sampling rate, ICA cleaned for eye movements/blinks + heart beat) where a subject listened to ~6min of continuous speech. We also have the corresponding acoustic envelope.

```py
# %% imports
import eelbrain as eb
import mne
import numpy as np
import joblib
from nilearn import plotting, datasets, surface

# %% define stuff
mri_path = '/mnt/obob/staff/preisinger/toolboxes/freesurfer/'

# %% load data
src_stuff = joblib.load("./data/subject01_sourcestuff_volume.dat")
data_meg = joblib.load("./data/subject01_meg.dat")
envelope = joblib.load("./data/subject01_envelope.dat")
```

Next, let's apply the inverse operator on our MEG dataset:

```py
# %% apply inverse operator (MNE)
print("Applying inverse operator (MNE)...")
snr = 3
lambda2 = 1 / snr ** 2  # = default value

stc = mne.minimum_norm.apply_inverse_raw(data_meg, src_stuff["inv"], lambda2=lambda2, method='MNE', pick_ori="vector", use_cps=False)
print("Done applying inverse operator (MNE)!")
```

Since eelbrain is an awesome toolbox, we can easily convert the MNE source time course object to an NDVar (a data class that eelbrain uses a lot):

```py
# %% convert MNE-python source time courses (stc) to eelbrain NDVar
meg_source = eb.load.fiff.stc_ndvar(stc, src='vol-10', subject='fsaverage', subjects_dir=mri_path, method='MNE', fixed=False, parc='aparc+aseg')
```

Now we are ready to compute a forward encoding model using the source-projected MEG data and the corresponding envelope. We use time lags between -100 and 600ms, split the data in 4 partitions and we also use selective stopping based on the _l2_ norm. With test=1, eelbrain does nested k-fold crossvalidation, i.e every partition serves as a test set once. We also save the model for later use, since estimating such models in source space can take quite some time.

```py
# %% train & test a forward encoding model
print("Training model...")
model = eb.boosting(meg_source, envelope, tstart=-0.1, tstop=0.6, basis=0.05, partitions=4, selective_stopping=1, test=1)
print("Done!")

# %% save the model for later use 
eb.save.pickle(model, "./data/subject01_fwd_model_volume.pickle")
# model = eb.load.unpickle("./data/subject01_fwd_model_volume.pickle")
```

Now we plot the TRF for every source, split by the two hemispheres:

```py
# %% plot TRF (whole brain)
fig = eb.plot.GlassBrain.butterfly(model.h)[0]
```

![](attachments/361465286/361465339.png)

We see two peaks at ~100ms and ~220ms, which are typical when using an acoustic envelope as a stimulus feature. Now we want to plot the first peak on a "Glassbrain", to see where it originates. :

```py
# %% plot specific TRF timepoint on a glassbrain
fig = eb.plot.GlassBrain(model.h.sub(time=0.1), vmax=0.025, interpolation="bicubic", title="100ms").plot_colorbar()
```

![](attachments/361465286/361465652.png)![](attachments/361465286/361465345.png)

We see that the activity mainly stems from right temporal sources (for cortical sources, eelbrain uses the [aparc atlas](http://ielvis.pbworks.com/f/1564069551/desikanKillianyAtlasKey.png)). Now we want to plot an averaged TRF from the superior temporal cortex. To get rid of the dipole orientations (which cannot be plotted here, but if you are interested in them see [here](https://doi.org/10.1016/j.neuroimage.2020.116528)), we take the vector norm over the three space dimensions. We also get rid of the first and last 50ms, which are usually regression artifacts.

```py
# %% plot averaged TRF (ROI)
labels = np.unique(model.h.source.parc) # get all atlas labels

# plot TRF averaged over superiortemporal sources
fig = eb.plot.UTS(model.h.sub(source=['ctx-lh-superiortemporal', 
                                'ctx-rh-superiortemporal'], time=(-0.05, 0.55)).norm("space").mean("source"), 
title="TRF (ROI)")
```

![](attachments/361465286/361465348.png)

That's a beautiful TRF! Now we also want to see the predictive power of it. First, let's plot all prediction accuracies on a Glassbrain:

```py
# %% plot prediction accuracies (whole brain)
fig = eb.plot.GlassBrain(model.r, vmax=0.1, interpolation="bicubic", title="Prediction accuracies").plot_colorbar()
```

![](attachments/361465286/361465350.png)![](attachments/361465286/361465349.png)

Again, we clearly see that the neural tracking is mainly happening in the right temporal cortex. On a single-subject level, a correlation of r=0.1 is also pretty good!

We can also project this data on a 3D surface brain using nilearn:

```py
# %% project volumetric data on a surface brain plot (using nilearn)

# get source space & convert to surface data
src = model.r.source.get_source_space()
img = eb.plot._glassbrain._stc_to_volume(model.r, src, mni305=True)

# get fsaverage dataset & surfaces
fsaverage = datasets.fetch_surf_fsaverage("fsaverage")
texture_right = surface.vol_to_surf(img, fsaverage.pial_right)

# surface plot
fig = plotting.plot_surf_stat_map(fsaverage.infl_right, texture_right, hemi='right', 
                                  bg_map=fsaverage.sulc_right, cmap="cold_white_hot", 
                                  colorbar=True, vmax=0.1, threshold=0.02)
```

![](attachments/361465286/361465354.png)
