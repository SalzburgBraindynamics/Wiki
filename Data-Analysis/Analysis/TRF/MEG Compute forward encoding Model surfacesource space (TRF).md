# MEG : 3a. Compute forward encoding model (TRF) - surface source space

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
src_stuff = joblib.load("./data/subject01_sourcestuff_surface.dat")
data_meg = joblib.load("./data/subject01_meg.dat")
envelope = joblib.load("./data/subject01_envelope.dat")
```

Next, let's apply the inverse operator on our MEG dataset:

```py
# %% apply inverse operator (MNE) 
print("Applying inverse operator (MNE)...")
snr = 3
lambda2 = 1 / snr ** 2  # = default value

stc = mne.minimum_norm.apply_inverse_raw(data_meg, src_stuff["inv"], lambda2=lambda2, method='MNE')
print("Done applying inverse operator (MNE)!")
```

Since eelbrain is an awesome toolbox, we can easily convert the MNE source time course object to an NDVar (a data class that eelbrain uses a lot):

```py
# %% convert MNE-python source time courses (stc) to eelbrain NDVar 
meg_source = eb.load.fiff.stc_ndvar(stc, src='ico-4', subject='fsaverage', subjects_dir=mri_path, method='MNE', fixed=True, parc='aparc')
```

Now we are ready to compute a forward encoding model using the source-projected MEG data and the corresponding envelope. We use time lags between -100 and 600ms, split the data in 4 partitions and we also use selective stopping based on the _l2_ norm. With test=1, eelbrain does nested k-fold crossvalidation, i.e every partition serves as a test set once. We also save the model for later use, since estimating such models in source space can take quite some time.

```py
# %% train & test a forward encoding model
print("Training model...")
model = eb.boosting(meg_source, envelope, tstart=-0.1, tstop=0.6, basis=0.05, partitions=4, selective_stopping=1, test=1)
print("Done!")

# %% save the model for later use 
eb.save.pickle(model, "./data/subject01_fwd_model_surface.pickle")
# model = eb.load.unpickle("./data/subject01_fwd_model_surface.pickle")
```

Now we plot the TRF for every source, split by the two hemispheres:

```py
# %% plot TRF (whole brain)
fig = eb.plot.brain.butterfly(model.h)
```

![](attachments/361465615/361465627.png)

We see two peaks at ~100ms and ~220ms, which are typical when using an acoustic envelope as a stimulus feature. Now we want to plot the first peak on a 3D brain, to see where it originates:

```py
# %% plot specific TRF timepoint on a 3D brain
fig = eb.plot.brain.brain(model.h.sub(time=0.1), vmax=0.04, title="100ms", smoothing_steps=25)
fig.plot_colorbar()
```

![](attachments/361465615/361465628.png)![](attachments/361465615/361465629.png)

We see that the highest activity is located at right temporal sources (eelbrain uses the [aparc atlas](http://ielvis.pbworks.com/f/1564069551/desikanKillianyAtlasKey.png)). Now we want to plot an averaged TRF from the superior temporal cortex. We remove the polarity by taking the standard deviation across sources (They are meaningful in principle, but not often interpreted in source space). We also get rid of the first and last 50ms, which are usually regression artifacts.

```py
# %% plot averaged TRF (ROI) 
labels = np.unique(model.h.source.parc) # get all atlas labels

# plot TRF averaged over superiortemporal sources
fig = eb.plot.UTS(model.h.sub(source=['superiortemporal-lh', 
                                'superiortemporal-rh'], time=(-0.05, 0.55)).std("source"), title="TRF (ROI)")
fig.set_ylim([0, .012])
```

![](attachments/361465615/361465630.png)

That's a beautiful TRF! Now we also want to see the predictive power of it. Finally, let's plot all prediction accuracies on a 3D brain:

```py
# %% plot prediction accuracies (whole brain)
fig = eb.plot.brain.brain(model.r, vmax=0.1, smoothing_steps=25)
fig.plot_colorbar()
```

![](attachments/361465615/361465634.png)![](attachments/361465615/361465631.png)

Again, we see that the neural tracking is mainly happening in the right temporal cortex. On a single-subject level, a correlation of r=0.1 is also pretty good!
