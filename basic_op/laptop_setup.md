# **2. Laptop Setup**
---
**[Global Protect](https://apps.microsoft.com/detail/9nblggh6bzl3?hl=en-us&gl=US)**<br>
The University of Salzburg offers a VPN service that lets you use your personal computer as if it were connected to the university network. To enable it, install the GlobalProtect Client for your operating system.
- Follow [this](https://im.sbg.ac.at/spaces/ITInfo/pages/42173841/VPN+-+Zugang+f%C3%BCr+Bedienstete#VPNZugangf%C3%BCrBedienstete-Englishinstructions) tutorial to set it up (scroll down for the English version).

**[OpenVPN](https://openvpn.net/community/)**<br>
The SCC (= Salzburg Collaborative Computing, i.e., the cluster of the University) requires you to install an additional VPN application called OpenVPN.
- Install it according to the instructions on the website.
- Ask [Thomas Hartmann](https://ccns.plus.ac.at/labs/auditory/members/thomas_hartmann/) to invite you as a pilot user of the SCC Pilot (he'll send you an OpenVPN profile file (with a `.ovpn extension`) and the username + password).
- Read [here](https://scc.plus.ac.at/pages/first-steps.html) on how to set it up correctly.
- You need to use OpenVPN if you want to access the cluster or the data server (read below).

**[Visual Studio Code (VSC)](https://code.visualstudio.com/)**<br>
Visual Studio Code is our main coding environment.
- Use it for all coding tasks except experiment coding.
- After installation, make sure to install the recommended extensions (Python, Jupyter, Git, etc.). You can read more on setting up VSC [here](xx hier annikas tutorial verlinken).

**[Python & Pixi](https://anaconda.org/conda-forge/pixi)**<br>
Python is the main language used in the lab for writing analysis code. Read [here](xxx code SOP tutorial verlinkenxxx) about our coding SOPs. 
- If you've done the previous step, you've already installed the Python extension in VSC. 
- After that, download Pixi and create your respective project-specific Python environment as explained [here](xxx-annika). 
- There is no need to install Python separately; Pixi and the VSC extensions handle this automatically.

**[MATLAB](https://de.mathworks.com/products/matlab.html)**<br>
MATLAB is used for experiment coding. Unlike Python, it must be installed separately on your laptop.
- The university provides MATLAB licences for people working here. Since the process of obtaining this licence involves several specific steps, it's best to follow [these](https://im.sbg.ac.at/spaces/ITInfo/pages/42174016/Allgemein+verf%C3%BCgbare+Software?preview=/42174016/373260885/TAH_Bedienstete_Einzelplatz.pdf#Allgemeinverf%C3%BCgbareSoftware-Matlab) instructions.
- Note: MATLAB scripts cannot be run in VSC, only viewed.
- MATLAB does not run on the SCC (i.e., the cluster).
- Use MATLAB (mainly) for coding experiments. See the [experiment tutorial](xxx) for more details.

**[GStreamer](https://gstreamer.freedesktop.org/download/#sources)**<br>
GStreamer is required for handling audio/ video streams in experiment presentation using Psychtoolbox and the o_ptb.
- Follow the operating-specific installation instructions provided on the [website](https://gstreamer.freedesktop.org/download/#sources).
- Read in [these](https://gstreamer.freedesktop.org/documentation/tutorials/index.html?gi-language=c) tutorials if you have further questions.

**[Psychtoolbox (PTB)](http://psychtoolbox.org/download.html)**<br>
Psychtoolbox (PTB) is a free MATLAB toolbox that lets you create and run experiments with precise timing. It can present visual and auditory stimuli, record responses, and interface with devices like eye-trackers, EEG, or MEG systems.
- Download and installation instructions can be found on the website 
- We recommend PTB versions up to and including 3.0.19.16, as these are free to use.

**[o_ptb](https://o-ptb.readthedocs.io/en/latest/install.html)**<br>
The o_ptb is a class-based objective for Psychtoolbox, making it easier to code experiments written by Thomas Hartmann.
- The documentation and installation guidelines can be found [here](https://o-ptb.readthedocs.io/en/latest/index.html).
<br>
<br>