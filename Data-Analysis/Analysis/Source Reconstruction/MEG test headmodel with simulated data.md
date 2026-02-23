# MEG : How can I test my head model with simulated data?

The following code snippets will allow you to double check the quality of your head model and your source reconstruction pipeline.

It can be done in three simple steps:

1. choose a grid point inside the brain and simulate its activity
1. project the simulated brain activity to the sensor space (using a forward model)
1. project the sensor-level activity back to the source space, using own head model and own analysis pipeline (inverse model)

The hope is that the reconstructed brain activity (cf. step 3) will be located very close to the simulated active source (cf. step 1).

Note, that source reconstruction can be biased towards the middle of the head when analyzing a single time series (a single "condition"). This can be circumvented by e.g. calculating the neural activity index, or by comparing data to a baseline period.

Here we will simply create two time series (two "conditions") with different amplitudes and the same source, which will be compared to each other.

For the simulation, we will need the following subject specific structures

1. source model (aka. grid) (=my_grid)
1. head model (=my_hdm)
1. leadfield (=my_lf)
1. meg data (=my_meg) including gradiometer information

---

## Simulating activity in the brain

Start from initializing obob_ownft and loading the subject specific structures:

```actionscript3
% initialize obob_ownft
cfg             = [];
cfg.package.svs = true;
obob_init_ft(cfg);

% load my structures
my_grid     	= load('S01_grid.mat');
my_hdm      	= load('S01_hdm.mat');
my_lf    		= load('S01_hdm.mat');
my_meg 			= load('S01_meg.mat');
```

```actionscript3
my_grid      	= ft_convert_units(my_grid,'m');
my_hdm       	= ft_convert_units(my_hdm,'m');
my_lf       	= ft_convert_units(my_lf,'m');
my_meg.grad 	= ft_convert_units(my_meg.grad,'m');
```

First, plot the source model (=grid points mesh) of your subject:

```actionscript3
figure; ft_plot_mesh(my_grid.pos(my_grid.inside, :))
```

Choose a grid point, where you wish to simulate activity. Use the data cursor (left mouse-button click on a grid):

![](attachments/27409621/27409627.jpg?effects=drop-shadow)

Then export the coordinates of the chosen grid point to the workspace (right mouse-button click on the grid → Export Cursor Data to to Workspace… → OK).

You can display once again the location of the chosen grid:

```actionscript3
figure;
hold on
ft_plot_vol(my_hdm,'facealpha',0.2,'edgecolor',[.8 .8 .8])
ft_plot_mesh(my_grid.pos(mygrid.inside, :))
ft_plot_mesh(cursor_info.Position,'vertexsize',40,'vertexcolor','red') % if you changed the name of the variable to which the grid coordinates were exported, you need to change here “cursor_info” to your own name
hold off
```

This is how it looks like:

![](attachments/27409621/27409629.jpg?effects=drop-shadow)

Now simulate activity in the chosen voxel. You can do it by creating a sinusoidal signal of a specific amplitude, phase, and frequency. You can also add some noise to the signal:

```actionscript3
cfg                 = [];
cfg.dip.pos         = cursor_info.Position; 	% coordinates of the chosen grid
cfg.dip.mom         = [1;1;1]; 					% orientation of the dipole
cfg.dip.frequency   = 3; 						% frequency of the simulated signal (Hz)
cfg.dip.phase       = 180; 						% phase of the simulated signal (radians)
cfg.ntrials         = 1000; 					% number of simulated trials
cfg.triallength     = 2; 						% length of each simulated trial (sec)
cfg.fsample         = 128; 						% sampling rate of the simulated signal
cfg.relnoise        = 2; 						% noise relative to the signal
cfg.headmodel       = my_hdm; 					% subject specific headmodel
cfg.grad            = my_meg.grad; 				% gradiometer structure 

% condition #1
cfg.dip.amplitude   = 75; 						% amplitude of the simulated signal #1
simulated_data1     = ft_dipolesimulation(cfg);

% condition #2
cfg.dip.amplitude   = 36; 						% amplitude of the simulated signal #2
simulated_data2     = ft_dipolesimulation(cfg);
```

The simulated_data looks exactly like an ordinary sensor-level MEG structure. It has as many trials as defined in cfg.ntrials in the code above.

---

## Inspecting simulated data in the sensor space

It could happen that the simulated data have different number and/or type of channels than your leadfield structure. Just make sure that this is not the case, otherwise you will run into troubles during source reconstruction. You can simply select a subsample of sensor based on the leadfield structure:

```actionscript3
cfg                 = [];
cfg.channel         = my_lf.label;
simulated_data1     = ft_selectdata(cfg,simulated_data1);
simulated_data2     = ft_selectdata(cfg,simulated_data2);
```

Now you can average across the trials and display the simulated data:

```actionscript3
% time-lock simulated sensor data
cfg                 = [];
cfg.vartrllength    = 0;
sim_tlk1            = ft_timelockanalysis(cfg,simulated_data1);
sim_tlk2            = ft_timelockanalysis(cfg,simulated_data2);
```

Select and plot magnetometers:

```actionscript3
cfg         		= [];
cfg.channel 		= ft_channelselection('megmag',sim_tlk1.grad);
cfg.layout  		= 'neuromag306mag';
figure;ft_multiplotER(cfg,sim_tlk1,sim_tlk2);
```

Magnetometers plotted:

![](attachments/27409621/27409630.jpg?effects=drop-shadow)

Select, combine and plot gradiometers:

```actionscript3
% select gradiometers
cfg         		= [];
cfg.channel 		= ft_channelselection('megplanar',sim_tlk1.grad);
plot_grad1  		= ft_selectdata(cfg,sim_tlk1);
plot_grad2  		= ft_selectdata(cfg,sim_tlk2);
 
% combine gradiometers (only for plotting!)
plot_grad_cmb1  	= ft_combineplanar([],plot_grad1);
plot_grad_cmb2 		= ft_combineplanar([],plot_grad2);
 
% plot gradiometers
cfg         		= [];
cfg.layout  		= 'neuromag306cmb';
figure;ft_multiplotER(cfg,plot_grad_cmb1,plot_grad_cmb2);
```

Gradiometers plotted:

![](attachments/27409621/27409631.jpg?effects=drop-shadow)

---

## Reconstructing simulated data in the source space

In the next step, you should use your own source reconstruction routine, which you created for analysis of the real MEG data.

Below is an example of calculating LCMV beamformers on single trials, as implemented in obob_ownft wrapper function. Note that single trial (and not time locked) data is used here as the input:

```actionscript3
cfg                 = [];
cfg.grid            = my_lf; % leadfields
data_source1        = obob_svs_beamtrials_lcmv(cfg, simulated_data1);
data_source2        = obob_svs_beamtrials_lcmv(cfg, simulated_data2);
```

Note, that the wrapper obob_ownft function created sensor-like data structure (a.k.a. virtual sensors).

Now, we will average the data across the trials:

```actionscript3
cfg                 = [];
cfg.vartrllength    = 0;
source_tlk1         = ft_timelockanalysis(cfg,data_source1);
source_tlk2         = ft_timelockanalysis(cfg,data_source2);
```

In the next step we calculate the difference between our two "conditions":

```actionscript3
source_diff     	= source_tlk1;
source_diff.avg    	= abs(source_tlk1.avg - source_tlk2.avg ./ source_tlk2.avg);
```

We can plot the calculated difference on a 2d plane, to see whether data makes sense:

```actionscript3
figure;imagesc(source_diff.avg); colorbar
```

The following figure pops up:

![](attachments/27409621/27409632.jpg?effects=drop-shadow)

On the image above, time is displayed on the x-axis and grid number on the y-axis. You can see that one of the grid points (here #587) reveals the strongest (sinusoidal) difference between the two conditions. The hope is that this is the grid point, where we put the simulated activity in the first place.

In the very last step we want to display our source reconstructed (simulated) data in a nice way. Wherefore the time-locked data should be reshaped, to look like data structure produced by ft_sourceanalysis function. Additionally, data can be already interpolated on an mri – a step that needs to be done before plotting the source data on an anatomical scan.

We need to provide here an mri structure and a grid structure. Just make sure that the two correspond to each other. They both must be either subject-specific structures, or templates (also in meters!):

```actionscript3
cfg             = [];
cfg.sourcegrid  = template_grid; 	% MNI grid, here 1.5cm
cfg.mri         = template_mri; 	% MNI template
cfg.parameter   = 'avg';
cfg.latency     = [source_diff.time(1) source_diff.time(end)]
cfg.downsample  = 2;
source_final    = obob_svs_virtualsens2source(cfg, source_diff);
```

Finally the simulated functional data can be displayed on an mri:

```actionscript3
cfg                 = [];
cfg.method          = 'ortho';
cfg.funparameter    = 'avg';
cfg.funcolormap     = 'jet';
cfg.interactive     = 'yes';
ft_sourceplot(cfg, source_final);
```

Let's have a look:

![](attachments/27409621/27409633.jpg?effects=drop-shadow)

It looks like it worked perfectly! The reconstructed activity is almost exactly in the same spot as the simulated activity.

It is recommended to do this kind of check with usage of a head model of every single subject.

---

Tutorial tested on MatLab version R2015a
