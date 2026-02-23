# MEG : How to calculate Granger Causality?

Granger causality gives you directed measures of connectivity between a seed region (voxel) and another region (voxel) as ingoing and outgoing connections to and from the seed region.

We start with transition of the cleaned and preprocessed data into source space

```actionscript3
%% beam single trials
cfg = [];
cfg.lpfilter = 'yes';
cfg.lpfreq = 90;
cfg.hdm = hdm;
cfg.grid = lf;
%cfg.regfac = '5%';
datasourceL = obob_svs_beamtrials_lcmv(cfg, data)


```

If you have not done already, we now split the data into conditions (this might be done at different stages of analysis).

```actionscript3
cfg = [];
cfg.trials = rev_trl; % take care that number of trials in conditions are equal
reverse = ft_selectdata(cfg, datasourceL);

clear datasourceL %not needed anymore, so we free memory


```

Next, we generate a structure for the to-be-calculated connectivity, as we will need it before in order to set the values at the seed-to-seed connectivity to Nan

```actionscript3
%%
cfg = [];
cfg.method = 'mtmfft';   %mtmfft for spectrum
cfg.taper = 'hanning';
conn_rev_ingoing = ft_freqanalysis(cfg,reverse);  
% only place holder to be filled below
conn_rev_outgoing = ft_freqanalysis(cfg,reverse);  


```

Next, you chose your seed voxel or parcel and reorder your data and rename the (additional) seed channel. This is important to ensure that the assigment of rows in the resulting grangerspectrum are always the same.

```java
%% we extract the seed from the data and append at at the end
% seed_ind: index of the voxel you choose as seed

cfg=[];
cfg.channel=reverse.label(seed_ind);
seed_chan=ft_selectdata(cfg, reverse);
seed_chan.label={'seed'};

cfg=[];
cfg.appenddim='chan';
rev_reorder=ft_appenddata(cfg, reverse, seed_chan) 
```

Next, we calculate Granger

```actionscript3
for ii=1:length(data_reorder.label)-1 % minus the added seed channel

   if ii==seed_ind
     conn_rev_ingoing.powspctrm(ii, :)=NaN;
     conn_rev_outgoing.powspctrm(ii, :)=NaN;
   else
    cfg = [];
    cfg.output = 'fourier';
    %cfg.frequency=[1 30];
    cfg.channel = ['seed', data_reorder.label(ii)]; % seed voxel x all other voxels
    cfg.method = 'mtmfft'; %mtmfft for spectrum
    cfg.tapsmofrq = 3;
    cfg.keeptrials = 'yes';
    cfg.pad=2;
    cfg.padtype='zero';
    fft_rev = ft_freqanalysis(cfg,reverse);
   
    
    cfg=[];
    cfg.method='granger';
    cfg.granger.sfmethod='bivariate';
    tmp_rev=ft_connectivityanalysis(cfg, fft_rev);

    conn_rev_ingoing.powspctrm(ii, :)=tmp_rev.grangerspctrm((1, :); 
    conn_rev_outgoing.powspctrm(ii, :)=tmp_rev.grangerspctrm((2, :); 
end
end

%!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!test your assignment with the following plots, outside the loop
cfg=[];
cfg.parameter='grangerspctrm';
figure; ft_connectivityplot(cfg, tmp_rev)

figure;
plot(tmp_rev.freq, tmp_rev.grangerspctrm(1, :), 'k'); hold
plot(tmp_rev.freq, tmp_rev.grangerspctrm(2, :), 'r');
```

As in Bastos et al. (2015) and Michalareas et al. (2016), when comparing two conditions, we normalize the data: e.g.  (ingoing- outgoing)/(ingoing+outgoing) and can then statistically compare the conditions.

If there is only one condition, you might need to flip the data and calculate Granger causality on the flipped data showing that the results are specific for the unflipped data.

GP might know more about this and consider consulting here: [http://www.fieldtriptoolbox.org/tutorial/salzburg](http://www.fieldtriptoolbox.org/tutorial/salzburg)
