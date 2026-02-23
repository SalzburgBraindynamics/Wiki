# MEG : Neural speech tracking analysis in source space using eelbrain

This is a tutorial on how to project MEG data to source space using MNE-Python and calculate forward encoding models (also known as temporal response functions/TRFs).

It starts with a first common step:

[1. Create a headmodel](1.-Create-a-headmodel_361465280.html)

Afterwards, you can choose between a surface or volumetric source space with either fixed or free-orienting dipoles. The choice depends on the question you have. Surface-based source spaces are restricted to cortical sources and are more commonly used in the literature. Volumetric source spaces also allows to look at subcortical brain regions. With surface source spaces, it is usually common to fixate the orientation of the dipoles perpendicular to the cortex. When using volumetric source spaces, the orientation of the dipoles is usually kept loose, i.e. let them freely orientate in a three-dimensional space (this allows to look at estimated current dipole orientation).

[2a. Calculate source ingredients - surface source space](2a.-Calculate-source-ingredients---surface-source-space_361465584.html)[2b. Calculate source ingredients - volume source space](2b.-Calculate-source-ingredients---volume-source-space_361465282.html)

Then, we calculate forward encoding models (boosting algorithm using eelbrain) using a speech envelope and we plot it in various ways. It is again split between surface and volume, since we need different parameters and plotting functions:

[3a. Compute forward encoding model (TRF) - surface source space](361465615.html)[3b. Compute forward encoding model (TRF) - volume source space](361465286.html)

---

Some prerequisites:

* Download this ZIP file which contains all the used data and also the scripts and plots: [https://plusacat.sharepoint.com/:u:/s/AG_Braindynamics/EQz05nVE0RVBrRc5xx-BWm8BU3XGIKVJJaO_xDCmI0mqng?e=5MKcha](https://plusacat.sharepoint.com/:u:/s/AG_Braindynamics/EQz05nVE0RVBrRc5xx-BWm8BU3XGIKVJJaO_xDCmI0mqng?e=5MKcha) (if the link stops working, it's also in the MS Teams channel "meg analysis" under files)
* To use the code in the tutorial, make sure you have a working Python environment with MNE, eelbrain, nilearn, joblib and numpy. The ZIP file contains an _environment.yml_ file to create such an environment (usage:_mamba env create -p ./.venv_) using [Miniconda](https://docs.conda.io/en/latest/miniconda.html) and [Mamba](https://mamba.readthedocs.io/en/latest/installation.html).
* You'll also need the fsaverage stuff from FreeSurfer, which you can access/copy from here: /mnt/obob/staff/preisinger/toolboxes/freesurfer

---

If you are not familiar with neural speech tracking, here are some must-read papers:

* Brodbeck, C., & Simon, J. Z. (2020). Continuous speech processing. Current Opinion in Physiology, 18, 25–31. [https://doi.org/10.1016/j.cophys.2020.07.014](https://doi.org/10.1016/j.cophys.2020.07.014)
* Brodbeck, C., Das, P., Gillis, M., Kulasingham, J. P., Bhattasali, S., Gaston, P., Resnik, P., & Simon, J. Z. (2022). Eelbrain: A Python toolkit for time-continuous analysis with temporal response functions. bioRxiv. [https://doi.org/10.1101/2021.08.01.454687](https://doi.org/10.1101/2021.08.01.454687)
* Crosse, M. J., Di Liberto, G. M., Bednar, A., & Lalor, E. C. (2016). The multivariate temporal response function (mTRF) toolbox: A Matlab toolbox for relating neural signals to continuous stimuli. Frontiers in Human Neuroscience, 10. [https://doi.org/10.3389/fnhum.2016.00604](https://doi.org/10.3389/fnhum.2016.00604)
* Crosse, M. J., Zuk, N. J., Di Liberto, G. M., Nidiffer, A. R., Molholm, S., & Lalor, E. C. (2021). Linear Modeling of Neurophysiological Responses to Speech and Other Continuous Stimuli: Methodological Considerations for Applied Research. Frontiers in Neuroscience, 15, 1350. [https://doi.org/10.3389/fnins.2021.705621](https://doi.org/10.3389/fnins.2021.705621)
* Gillis, M., Van Canneyt, J., Francart, T., & Vanthornhout, J. (2022). Neural tracking as a diagnostic tool to assess the auditory pathway. Hearing Research, 426, 108607. [https://doi.org/10.1016/j.heares.2022.108607](https://doi.org/10.1016/j.heares.2022.108607)
* Obleser, J., & Kayser, C. (2019). Neural Entrainment and Attentional Selection in the Listening Brain. Trends in Cognitive Sciences, 23(11), 913–926. [https://doi.org/10.1016/j.tics.2019.08.004](https://doi.org/10.1016/j.tics.2019.08.004)

---

Please keep in mind that we use the boosting algorithm in this tutorial!
