# MEG : Chapter 4c - Train, find the component automatically

Ideally, blink and ECG components are continuous and easy to identify. As the train is not stationary, it is more difficult to find. Usually, even after maxfilter, there are some residuals of the train (16.7 Hz, the exact frequency in your data depend on your sampling rate).

To identify the component capturing the train is easy if you look at trials that contain them.

![](attachments/81035677/81035681.png)![](attachments/81035677/81035685.png)

However, only few trials might show this component and the topography of the train component is not consistent.

Therefore, if you have many trials and the train does not occur in the beginning but towards the end, it might be time-consuming to scroll through all trials in order to find it.

To avoid this, you can use the function **obob_find_comp_pattern** (in the utilities package of obob_ownft) that runs an fft over the components and tells you which one has a maximum around a specified frequency.

All default setting are targeted to the train artifact, but you can also search for e.g. line noise or other frequencies of your choice.

You can use it as following:

```java
cfg=[];
[comp_chan, comp_trl] = obob_find_comp_pattern(cfg, comp)

```

**comp_chan** is the number of the component you are looking for (theoretically there can be more than one) and **comp_trl** tells you in which trials the specified frequency peaks.

If the  number of components is surprisingly high, it might be the rare case that no train was recorded and therefore many components are equally close to the 16.7. Hz maximum. Or alternatively, that (perhaps due appending fif-files and head movement), actually, many components carry the train artifact.

It is advised that you validate the results in the data browser. Just go to the channel and trial of interest and check if the component captured the train.

```java
%%
cfg=[];
cfg.layout='neuromag306mag.lay'
cfg.compscale = 'local';
ft_databrowser(cfg,comp);
```

This is the code of the function:

```java
function [comp_chan, comp_trl] = obob_find_comp_pattern(cfg, comp)
% Use as
%    [comp_chan, comp_trl] = obob_find_comp_pattern(cfg, comp)
%
% this functions searches for the components for which the power is within
% a certain range (e.g. train artifcat)
%
% input:
%   component structure (output of ft_componentanalysis)
% output:
%   the component number of the defined fft pattern
%   trial numbers showing this pattern strongest
%
% optional:
%   cfg.freq_peak = peak frequencies of you pattern (default for train [16.5 16.5)
%   cfg.win_width = frequencies range around peak frequency (default = 2)
%   cfg.win_res   = frequency resolution (default = 0.2)
%   cfg.thres     = number, you can set the threshold for pattern (default=0.7 of maximum)
%   cfg.trl_list  = 'yes' or 'no', list the trials that show the pattern  (default = 'yes')
%   cfg.normalize = 'yes' or 'no', In our experience so far (very limited): for
%                   train artefacts, set cfg.normalize = 'no' (default= 'no')
%
% Copyright (c) 2019, Anne Hauswald & Thomas Hartmann
%
% This file is part of the obob_ownft distribution, see: https://gitlab.com/obob/obob_ownft/
%
%    obob_ownft is free software: you can redistribute it and/or modify
%    it under the terms of the GNU General Public License as published by
%    the Free Software Foundation, either version 3 of the License, or
%    (at your option) any later version.
%
%    obob_ownft is distributed in the hope that it will be useful,
%    but WITHOUT ANY WARRANTY; without even the implied warranty of
%    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
%    GNU General Public License for more details.
%
%    You should have received a copy of the GNU General Public License
%    along with obob_ownft. If not, see <http://www.gnu.org/licenses/>.
%
%    Please be aware that we can only offer support to people inside the
%    department of psychophysiology of the university of Salzburg and
%    associates.

% set the defaults
freq_peak = ft_getopt(cfg, 'freq_peak', [16.5 16.5]);
win_width = ft_getopt(cfg, 'win_width', 2);
win_res = ft_getopt(cfg, 'win_res', 0.2);
thres = ft_getopt(cfg, 'thres', 0.7);
do_list = ft_getopt(cfg, 'trl_list', 'yes');
do_norm = ft_getopt(cfg, 'normalize', 'no');


cfg = [];
cfg.method = 'mtmfft';
cfg.foi = [(freq_peak(1) - win_width) : win_res : (freq_peak(2) + win_width)];
cfg.keeptrials = 'yes';
cfg.tapsmofrq = 1;
cfg.output = 'pow';

fft = ft_freqanalysis(cfg, comp);

if strcmp(do_norm, 'yes')
    [z, mu, sigma]=zscore(fft.powspctrm, 0, 2);
    fft.powspctrm=z;
end

cfg = [];
cfg.avgoverrpt = 'yes';
cfg.frequency = freq_peak;
cfg.avgoverfreq = 'yes';
tmp = ft_selectdata(cfg, fft);

comp_chan = find(tmp.powspctrm >= (max(tmp.powspctrm)) * thres);

if strcmp(do_list, 'yes')
    
    cfg = [];
    cfg.avgoverchan = 'yes';
    cfg.frequency = freq_peak;
    cfg.avgoverfreq = 'yes';
    tmp2 = ft_selectdata(cfg, fft);
    
    comp_trl = find(tmp2.powspctrm >= (max(tmp2.powspctrm)) * thres); %
    
else
    comp_trl = 'no';
end
```
