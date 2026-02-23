# MEG : Chapter 4a - A note on preprocessing

From the [chapter 4](Chapter-4---Preprocessing-and-epoching_21140178.html) I assume you know how to read in your data and did that. Great!

First, I want to highlight that there is [no standard way how to preprocess / filter the data](https://sccn.ucsd.edu/pipermail/eeglablist/2015/010012.html). Here, I want to focus on filtering as this is a critical step in the preprocessing (see links to the sources provided in the text).

Often a 1-Hz hp filter has been applied (I myself did it as well in some of my studies!). However, [evidence](http://ac.els-cdn.com/S0165027012002361/1-s2.0-S0165027012002361-main.pdf?_tid=2859dff4-7389-11e7-868e-00000aab0f6c&acdnat=1501242039_c9bde4dd06ab08fd548a55af4137edfe) is increasing that certain types of filters can significantly distort your data (especially if you are interested in ERFs or phase). I suggest to invest some time and read about which filter to apply in your specific case and how to apply these filters to your data before starting the preprocessing.  You may want to start looking for relevant sources e.g. [here](https://sccn.ucsd.edu/wiki/Firfilt_FAQ) (including the section 'References and recommended readings'; e.g. the book of S. Luck (2014) is very helpful). Also check the current literature in your research field to see what is common practice there (check papers from different labs of good reputation) and ask your colleagues. Also, you may want to check out discussion lists from [eeglab](https://sccn.ucsd.edu/pipermail/eeglablist/2017/012303.html) or [fieldtrip](https://mailman.science.ru.nl/pipermail/fieldtrip/2015-September/009658.html).

In general, note that [applying filters on continuous data](http://www.sciencedirect.com/science/article/pii/S0165027014002866) is recommended. Let us assume you decided to apply a high-pass filter of 0.1 Hz (compared to 1 Hz). This brings the first challenge. If the MEG data were sampled with a rate of 1000 Hz the preprocessing can take very long (up to 25 min and more for only 1 block of 1 participant). A workaround could be to first apply a lowpass filter of your choice (also to [avoid aliasing](https://en.wikipedia.org/wiki/Anti-aliasing_filter); see also Luck (2014)) and then downsample your data (to 250 Hz; good choice for most cognitive experiments according to Luck (2014); except you are interested in very early sensory responses such as < 20 ms).

So, let us lowpass filter the data first by using a 35-Hz lowpass filter (kaiser-windowed sinc FIR filter). Why such a low lowpass filter? In this particular case I am not interested in frequencies higher than 35 Hz. Lowpass-filtering also reduces noise in the data. Just by applying this filter we get rid of muscle activity (>40 Hz) and line noise (~50 Hz). Note that I do not use the default filter by fieldtrip but the Kayser filter (also used by Philipp Ruhnau, Andreas Widmann, etc). However, there might be even more appropriate ones for your specific case (e.g. [Gaussian or Bessel filters to avoid or reduce ringing artifacts](http://www.sciencedirect.com/science/article/pii/S0165027014002866)). For kaiser-windowed sinc FIR filter you need to specify also the transition width (which is less crucial for a low pass filter compared to a high pass filter, though it should not be too high) and the filter type 'firws'.  You can also plot the [filter response](http://www.sciencedirect.com/science/article/pii/S0165027014002866) to estimate whether the settings of the choosen filter are appropriate. For recommendations on how to design filters check out e.g. [this source](http://home.uni-leipzig.de/biocog/widmann/eeglab-plugins/firfilt.pdf).

Note that if you apply this filter, make sure to save the output, as it is[good practice in reporting MEG research](https://www.ncbi.nlm.nih.gov/pubmed/23046981) (e.g. in publications) to define the specific filter settings.

Applying this filter would give you the following output:

After lowpass filtering the data, we are ready to downsample the data. Note that downsampling continuous data might be better than downsampling epoched data. This is because of the way how it is implemented in fieldtrip. A discussion with implications on that you will find [here](https://mailman.science.ru.nl/pipermail/fieldtrip/2015-January/008879.html).

Now, we can finally apply the highpass filter. Note that applying a 0.1-Hz hp-filter takes now less than 1 minute (instead of > 20 min; see above). Super awsome, isn't it?!

Also for the hp, we apply the  Kaiser-windowed sinc FIR filter. Here the choice of the transition width is a critical one! What do we need to consider?  The transition width is at max twice the highpass. Also note that the cut off frequency minus the transition band width / 2 must not be < 0. If we choose a 0.1 Hz high pass with a transition width of  0.1 all criteria are fullfilled: 0.1 – 0.1 / 2 = 0.05. Here you can find a more deteailed '[recipe](http://home.uni-leipzig.de/biocog/widmann/eeglab-plugins/firfilt.pdf)'.

Again, remember to save the output from hp-filtering. For the current case, this were:

Now, we are interested in epoching the data. You know from [Chapter 4](https://im.sbg.ac.at/display/MEG/Chapter+4+-+Epoch+data) how this works (see section 'Define your trials based on triggers value'; resulting in **cfg_trials**).

You probably know what the **cfg_trials** structure contains. If not, check it out! The first column contains the sampling points defining when the defined epochs start. The second column contains the sampling points defining when the defined epochs end. The third column contains the sampling points defining the[offsets](http://www.fieldtriptoolbox.org/reference/ft_redefinetrial). As we downsampled the continuous data, the sampling points in the **cfg_trials** structure do not match with the sampling points in the continous data. To get the epochs we want, we need to adjust the sampling points in **cfg_trials.**

How? We simply devide them by the ratio old_sampling-frequency/ new_sampling-frequency = 1000 / 250 = 4.

So, do the following:

Now, you can cut the data into epochs as documented in [Chapter 4](https://im.sbg.ac.at/display/MEG/Chapter+4+-+Epoch+data):

So, you may want to check how your data look like after applying the filters and/or after epoching. You can do this via ft_databrowser. Make sure to use the correct configuration (see comments below) and the data of interest.

Let us assume you want to check the data on which a 35-Hz lp- and a 0.1 Hz hp filter was applied before they were segmented into epochs. When doing so you may observe something like this (note that the plot below does NOT stem from the data set used in Chapter 4):

![](attachments/23859824/23859899.png)This is one epoch from -1 to +5 sec showing the data on all channels. We can observe huge slow drifts on some of the channels (mainly magnetometers). They can be explained by using the hp-filter of 0.1 Hz (which does not eliminate the drifts). For gradiometers the slow drifts are less prominent.
