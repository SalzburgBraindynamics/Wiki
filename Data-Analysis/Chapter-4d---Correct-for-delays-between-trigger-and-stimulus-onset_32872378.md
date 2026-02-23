# MEG : Chapter 4d - Correct for delays between trigger and stimulus onset

You should measure your delays between triggers and stimulus onset with the blackbox prior to really running the experiment. Ask Fred for help. Once you determined your delays, you can use that information to correct your data.

The physical delay of auditory stimulation due to the tubes is 16 ms.

The code you can use for this is ft_redefinetrial with cfg.offset which offsets the data in relation to the timeaxis, i.e. time point zero. This means if you e.g. have the 16 ms delay of your stimulus arrival, you would want to shift your data -16 ms, i.e. data at time point=16 ms is redefined as data at time point=0.

**IMPORTANT**: 'minus' if stimulus arrives after trigger. **IMPORTANT**: provide sampling points and not timepoints. That is, if you sampled at 1000 Hz, then the number of milliseconds corresponds to the number of samples, but if you sampled higher or downsampled you've to modify this number accordingly, i.e. for the 10kHz Fs this will be `cfg.offset=-164` (the real delay was 16.4 ms).

**NB**: if your dataset has been collected before September 2017, then the delay back then was about 24.5 ms, please beware when you analyse old data. If you are still on Trento data just ask me (GD), or someone in Trento ... we've to see ...

```java
cfg=[];
cfg.offset=-16;
% IMPORTANT: 'minus' if stimulus arrives after trigger. 
% IMPORTANT: provide sampling points an not timepoints.
data_offset=ft_redefinetrial(cfg, data)
```

Note, that due to the shift the data now has 'pre-stim' time points, meaning that data.time(1)=-16 and data.time(17)=0.

If you want the data to start at time point zero, you have to redefine the trial using a start sample (here 17) and an end sample.
