# MEG : Time frequency (TF) analysis

TODO - Work in progress.

**The wiki would highly benefit from experts' knowledge on TF analysis, especially on things like what originates from experts' own experience and/or based on what experts read when developing their analysis pipeline and what parameters turned out to be well suited for analysing e.g. alpha power and other frequency bands. A nice overview of parameters to use for the respective approach could be very helpful (maybe in form of a table) and some explanation why choosing those parameters. Sources where the knowledge comes are also very valuable. They will e.g. be of great insight for the naive ones to read more in depth about it.**

**The wiki focuses now only on power. However, in case there is also knowledge in the group how to analyse phase this might also be valuable information to put in the wiki.**

**The wiki would, of course, also benefit from non-experts' feedback, e.g.  to get an idea what becomes clear from the TF tutorial in its current state and what should become clearer / is missing.**

**Experts' knowledge and non-experts' feedback will most likely help us all to store and share knowledge on TF analysis in a very efficient way.**

The text originates from what TH has written in the CIMeC wiki (see[here](https://wiki.cimec.unitn.it/tiki-index.php?page=TutCIMEC06)) and from the fieldtrip tutorial (see [here](http://www.fieldtriptoolbox.org/tutorial/timefrequencyanalysis)).

### [](#Timefrequency(TF)analysis-) [Why and how do we average?](#Timefrequency(TF)analysis-Whyandhowdoweaverage?) [But, what do you mean by oscillations?](#Timefrequency(TF)analysis-But,whatdoyoumeanbyoscillations?) [But why would I want to care about oscillations?](#Timefrequency(TF)analysis-ButwhywouldIwanttocareaboutoscillations?) [Lets analyse some data](#Timefrequency(TF)analysis-Letsanalysesomedata)  [Planning the function](#Timefrequency(TF)analysis-Planningthefunction)   [The header, loading, resampling, cutting and interpolating the data](#Timefrequency(TF)analysis-Theheader,loading,resampling,cuttingandinterpolatingthedata) [Doing the time-frequency transform](#Timefrequency(TF)analysis-Doingthetime-frequencytransform)  [Single taper (hanning):](#Timefrequency(TF)analysis-Singletaper(hanning):) [Multitaper](#Timefrequency(TF)analysis-Multitaper) [Morlet wavelets](#Timefrequency(TF)analysis-Morletwavelets)   [FAQs](#Timefrequency(TF)analysis-FAQs)  [1.) Do I have to take special care about a non-zero DC component that might be present in the time domain data?](#Timefrequency(TF)analysis-1.)DoIhavetotakespecialcareaboutanon-zeroDCcomponentthatmightbepresentinthetimedomaindata?) [2.) Do I have to take special care about slow frequency drifts that might be present in the time domain data?](#Timefrequency(TF)analysis-2.)DoIhavetotakespecialcareaboutslowfrequencydriftsthatmightbepresentinthetimedomaindata?)

### Why and how do we average?

**Why**: "Because otherwise I do not see anything!". Well, you are right. But let us take a closer look at what averaging does to our data:

We can also rephrase what we want to do: Let's keep the signal and throw away the noise.

So, we assume that the data we recorded is a mixture of the true signal and some random noise. We also assume that the noise is really and literally random while the actual signal is the exact opposite: it is exactly the same in each and every trial: It starts at the exact same latency, has the exact same amplitude, has the exact same duration. For every trial of each condition.

You might agree that these are quite strong assumptions. The most important consequence of this assumption is that the brain activity before stimulus presentation has no impact on the post stimulus activity. You could summarize this approach as "Something appears out of Nothing", i.e. the evoked potential is something entirely new and unrelated to what has happened before the event causing its generation.

However, these assumptions are not necessarily true. Especially as cortical oscillations recorded at the scalp are considered more and more to represent meaningful information about long and shortrange communication in and between brain areas. These oscillations are more or less present all the time. Processing of an incoming stimulus might reset the so-called phase of the oscillation, thereby aligning it to the stimulus onset. When this happens, the evoked approach retains this activity. If, however, the phase does not change but the stimulus only affects the amplitude of the oscillation, averaging completely misses the effect.

![](https://wiki.cimec.unitn.it/tiki-download_file.php?fileId=329&display)

So, the punch line is if we want to see all the oscillatory responses to our stimulus, we would need to change something in our averaging procedure.

### But, what do you mean by oscillations?

If you have never worked with EEG or MEG, the concept of oscillations might be a bit strange for you. However, oscillations are extremely prevalent in EEG/MEG data. You can actually see them with the naked eye like Berger did in his first EEG recording as well as Adrian and Matthews back in 1934:

![AlphaEyesOpenClosed](https://wiki.cimec.unitn.it/tiki-download_file.php?fileId=330&display)

These are the so-called alpha waves or alpha oscillations. You see that they seem to react to what the subject is doing: Whenever he closes his eyes, more of these alpha waves appear, i.e. the amplitude increases. When he opens his eyes, you see less of them, i.e. the amplitude decreases.

If you were to analyze the above data, what you would want to know is the amplitude of the oscillations over time. There are a bunch of different methods around. But as we normally want to not only look at one frequency but at many of them, there are two principle methods that are used nowadays. Wavelets and Windowed FFT. Both methods have in common that they divide your data into small overlapping pieces of some milliseconds and calulate the amplitude of the oscillations inside each window. The length of these windows depends on the frequency because slower frequencies tend to change their amplitude slower than higher frequencies. I will not go into the details of how the two methods calculate the amplitude here.

Besides the amplitude, we need one more parameter to describe an oscillation. This one is called the "phase" and describes the temporal shift.

![](https://wiki.cimec.unitn.it/tiki-download_file.php?fileId=331&display)Illustration of different phase shifts. (C) by [http://www.allaboutcircuits.com/](http://www.allaboutcircuits.com/)

The above picture gives you an illustration of what phase and, more importantly, phase shift between two oscillations mean. Now, imagine that you would average the A and B waveforms in each of the panels. You can easily see that although they all have the same amplitude, the amplitude of the average would be highest in the example of zero phase shift, somewhat lower in the 90 degree phaseshift and 0 in the 180 degree phase shift example. Now imagine that you have many of these waveforms and the phase is distributed randomly. If you average those, you would only see a flat line.

So, here is the trick: First transform the data to the time-frequency domain for each trial and average those later. Thereby you get rid of your phase problem.

### But why would I want to care about oscillations?

There are a lot of explanation for this. I will only give the short answer here. If you want to elaborate more on that, I would like to refer you to the excellent book by Buzsaki called "Rhythms of the Brain". The short story goes like this: The human cortex contains like 20 billion neurons. Each of these neurons is connected to something like 10.000 other neurons. Most of these connections are to the neighbors but some of them are very long. Lay back and think about these numbers again. They are huge! Especially the amount of connections involved here is tremendous. When we look at brain functions with the EEG or MEG we are not interested what one of these 20 billion neurons is doing. One of them is actually totally insignificant to our signal. We are measuring a few 10.000 of them, maybe 100.000. And they **must do the same thing**! This is really important and bears some large scale implications. If these neurons just fire randomly, all the electric and magnetic fields that they would create would cancel each other out and we measure nothing. Literally. If you look at this from the other end, it means that only if coordinated firing of neurons, i.e. synchronization between lots of them, is a fundamental principle, we can see something meaningful with these methods. Fortunatly for us, this is actually the case. But, how can synchronization in a system of billions of neurons with thousands of connections work? Well, fortunatly for us, the brain seems to be wired in a pretty particular way, forming a non-linear system. (You do not have to understand here what that term means. It is the butterfly -> hurricane stuff....). And these systems have the _inherent_ property that they oscillate. And oscillations in turn are a great way to synchronize elements of a system.

If I have lost you here, here is the ultra short story: Oscillations organize the mess 20 billion highly interconnected neurons would create otherwise. Ok?

### Lets analyse some data

#### Planning the function

You will see that time frequency analysis resembles the timelocked analysis we did in the previous chapter. There are, however, some differences. For example, we do not filter the data because it simply does not make any sense. So, here is what our function will do:

1. Load the data
1. Resample the data. We will resample to a slightly higher sampling frequency of 300Hz here.
1. The interval we need for the analysis is bigger than that for the timelocked analysis. We will cut the data between -1 to 2 seconds.
1. Interpolate missing channels.
1. Do a seperate ft_freqanalysis for each condition.
1. Combine the planar gradiometers.

### The header, loading, resampling, cutting and interpolating the data

As this is stuff you already know from the previous chapter(s), we can be quick here:

```java
% first things first: load the data...
load(fullfile(indir, subject));

% resample
cfg = [];
cfg.resamplefs = 200;
cfg.detrend = 'no';

data = ft_resampledata(cfg, data);

% and we also do not need all the data...
cfg = [];
cfg.toilim = [-1 2];

data = ft_redefinetrial(cfg, data);

% now we have to interpolate the data to make up for the rejected channels.
cfg = [];
cfg.interp = 'nearest';

data_interp = obob_fixchannels(cfg, data); % just copied from the tutorial - different now?
```

### Doing the time-frequency transform

Time-Frequency transforms in FieldTrip are done using the function _ft_freqanalysis_. If you look at the help file of the function you see that it has lots and lots of options and there are a few choices you can do here. The basics of each time-frequency transform are the same: You take a small window of your data and analyze the amplitude of one frequency. Then you take the next window. You do this separatly for every frequency you want to analyse because it is a very good idea to use large windows for low frequencies and small windows for high frequencies. On the other hand, signals tend to get more _broadband_ the higher the frequency is. So, when you analyze low frequencies you want to analyze only this one, if you go for higher frequencies, e.g., 40Hz, you would probably want to include 35-45Hz as well. The other problem is that the algorithms you use for analyzing the amplitude of the signal need the signal to be zero at the start and end of the window and it should not be so jumpy.

FieldTrip offers basically two different methods, _wavelet_ and _mtmconvol_ that deal with these issues in a different way. They do have some strengthes and weaknesses but should basically produce the same output. We will use the _mtmconvol_ method here. This method is quite simple: It takes one of the windows that I have mentioned above. Then it "tapers" the data within this window. Basically, this means that it multiplies the data window with a function that is zeros at the beginning and the end and 1 in the middle. If you want to know more about window functions, click [here](http://paulbourke.net/miscellaneous/windows/).

However, when you multiply your data with a window function, you also modify it in the frequency domain. I will not go into too much detail here (for the nerds: remember that multiplication in the time domain = convolution in the frequency domain and skip this paragraph ;-) ). Basically what happens is this: Peaks get broader. If you have a peak at only one specific frequency, it will spread out a little to the neighbours. This might seem bad at the beginning but is actually pretty great. An FFT offers only a limited frequency resolution which depends on the amount of time you apply it on. For instance, if you have windows of 1 second, your resolution will be 1Hz. If your real oscillation is at 10.5Hz, you will actually see two peaks, one at 10Hz, the other at 11Hz. The same oscillation at 10Hz would only produce a peak at 10Hz but the amplitude would seem to be doubled because in the former case, the power is split between 10Hz and 11Hz. So, the 10.5Hz oscillation would seem of less power than the 10Hz oscillation, although they are equally strong. If you smear both oscillations a bit using tapers, they show up as having the same amplitude.

This effect works quite nicely for lower frequencies (like up to 20Hz). But for higher frequencies, as I said before, you want to include more and more frequencies around the center. This cannot be done anymore by only tapering the data once but you need to use the so called multitaper approach to achieve this.

#### **Single taper (hanning):**

```java
cfg = [];
cfg.method = 'mtmconvol';
cfg.output = 'pow'; 
cfg.taper = 'hanning'; % The window function
cfg.foi = 4:1:30; % The frequencies. we start at 4Hz and go up to 30Hz in 1Hz steps
cfg.toi = -1:.03:2; % Timebins. We start 1s before stim onset and take one bin every 30 ms until we reach 2s post stim
cfg.t_ftimwin = 4./cfg.foi; % The size of the window. In this case, every window is 4 cycles long.

% now we execute it for every condition....
for i = 1:length(all_conds)
  cur_condition = conditions.(all_conds{i});
  cfg.trials = cur_condition.trls;
  
  data_tfr.(all_conds{i}) = ft_freqanalysis(cfg, data_interp);
end %for




```

#### **Multitaper**

If you want to analyse e.g. gamma, you may want to apply multitaper. For more details and background info, see e.g. the [ft-tutorial](http://www.fieldtriptoolbox.org/tutorial/timefrequencyanalysis).

```java
cfg = [];
cfg.method = 'mtmconvol';
cfg.output = 'pow'; 
cfg.taper = 'dpss'; % The window function
cfg.foi = 30:2:200; % The frequencies. we start at 30 Hz and go up to 200 Hz in 2 Hz steps
cfg.toi = -1:.03:2; % Timebins. We start 1s before stim onset and take one bin every 30 ms until we reach 2 s post stim
cfg.t_ftimwin = 4./cfg.foi; % The size of the window. In this case, every window is 4 cycles long.
cfg.tapsmofrq = cfg.foi/10; % The amount of smoothing per frequency. In this case, it is a 10th of the frequency.
```

#### Morlet wavelets

An alternative to calculating TFRs is to use Morlet wavelets. The approach is equivalent to calculating TFRs with time windows that depend on frequency using a taper with a Gaussian shape. The commands below illustrate how to do this. One crucial parameter to set is cfg.width. It determines the width of the wavelets in number of cycles. Making the value smaller will increase the temporal resolution at the expense of frequency resolution and vice versa. The spectral bandwidth at a given frequency F is equal to F/width*2 (so, at 30 Hz and a width of 7, the spectral bandwidth is 30/7*2 = 8.6 Hz) while the wavelet duration is equal to width/F/pi (in this case, 7/30/pi = 0.074s = 74ms). (Info taken from [here](http://www.fieldtriptoolbox.org/tutorial/timefrequencyanalysis)).

```java
cfg = [];
cfg.channel    = 'MEG';	                
cfg.method     = 'wavelet';                
cfg.width      = 7; 
cfg.output     = 'pow';	
cfg.foi        = 1:2:30;	                
cfg.toi        = -0.5:0.05:1.5;		              
TFRwave = ft_freqanalysis(cfg, dataFIC);
```

### FAQs

#### 1.) Do I have to take special care about a non-zero DC component that might be present in the time domain data?

The short answer is no. If you're applying the default option cfg.polyremoval = 0 in ft_freqanalysis, the zero-order polynomial is removed, which is equal to demeaning. That is, you don't have to demean the data separately via ft_preprocessing. More details on that issue you can find [here](http://www.fieldtriptoolbox.org/faq/why_does_my_tfr_look_strange).

#### 2.) Do I have to take special care about slow frequency drifts that might be present in the time domain data?

Yes. The problem with a low-frequency component is that it leaks into the estimates of all time-frequency points in a variable (but patterned) way. The reason why this happens is related to the fact that none of the tapered basis functions (i.e. windowed sine and cosine waves of increasing frequency) are exactly orthogonal to this slow drift. A solution to this problem is to detrend or high-pass filter your data prior to calling ft_freqanalysis.

```java
cfg = [];
cfg.detrend = 'yes';
data = ft_preprocessing(cfg, data);
```

Instead of detrending or applying an appropriate highpass is the following solution: You could set cfg.polyremoval = 1 (NOT the default). This option removes BOTH the zero-order (demeaning; see FAQ number 1) and first-order (detrending) polynomial. More details on that issue you can find [here](http://www.fieldtriptoolbox.org/faq/why_does_my_tfr_look_strange_part_ii).
