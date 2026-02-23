# MEG : Chapter 5 - ERF

You now have cleaned data and have interpolated bad channels.The next step in our pipeline is to calculate evoked fields.

Even if you have no interest whatsoever for ERFs / ERPs they constitute in most experiments a good sanity check with regards to data quality. Since it is ridiculously easy and quick to do, just do it.

This is how you do it:

Note that you can add preprocessing options (all that listed in ft_preprocessing) also in this step by adding a 'preproc' to the cfg. When leaving this out -i.e. the cfg is empty- Fieldtrip simply calculates the average. You will notice the difference in speed. In this example I added some low-pass filtering, as is common in the ERF / ERP world.

You can now take a look at the sensor time-series layout of the magnetometers:

Note the option 'interactive'. This means again that you can choose to plot sensors and within these sensor plots time periods to get topographies.Here an overview of the magnetometer data:

![](attachments/21140196/21140515.png)

Topography 0-150ms using Magnetometers:

![](attachments/21140196/21140516.png)

Note the clear dipolar field that you get using magnetometers. With the background information of a auditory stimulation you can assume an active source in between the two maxima. Beware that in the magnetometer display the source is most unlikely there where you have the maximum fields!!! This is important in being aware that you shouldn't interpret in anyway anatomical even if speaking of "frontal" sensors.

A better estimation of potential locations of sources is given by gradiometers (at the cost of being sensitive to mostly superficial sources). For this purpose we need to combine the information from the two gradiometers in order to obtain the planar gradient (basically the vector in x- and y-direction). This is, however, only necessary in sensor space (before statistics: see e.g. [here](https://mailman.science.ru.nl/pipermail/fieldtrip/2015-November/009787.html) ; and before plotting, see e.g. [here](http://www.fieldtriptoolbox.org/tutorial/eventrelatedaveraging))!

Look at the data structure. You will see that you still have the 102 magnetometers, but that the info from the 204 gradiometers is now combined to 102 planar gradients. This means that you now have 204 sensors in the data structure. Since the magnetometers are included all viewing options given above can be applied to this data structure as well.

Let's take a look at the multiplot and topography as above but now for the planar gradiometers:

![](attachments/21140196/21140517.png)

Topography 0-150ms using Gradiometers:

![](attachments/21140196/21140518.png)

Note the fundamental difference to the magnetometer depiction. Where the source is probably located according to the magnetometers i.e. in between the maxima / minima, the gradiometers has its fattest blob. This nicely illustrates the advantage of gradiometers as they the probably source is right beneath the maximum amplitudes. This means that careful anatomical inferences at the sensor level can be done better using gradiometers (e.g. frontal source ... but not Brodman area bla!). The cost is -to repeat it again- lack of depth sensitivity. So both information are valuable (this is emphasized since many published studies remove magnetometer information) and when exploring your data then look at both.
