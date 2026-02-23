# MEG : After entering the lab

Switch on the light in the MEG chamber: make sure that the Brightness is set in between min and max (center position!) for healthy participants!

* **after liquefaction is done: tilt MEG from liquefaction position (25°) to measurement position (0°, 60° or 68°(our standard))******

The MEG can be set to four fixed positions, driven by a linkage: three measurement positions and one liquefaction position.The three measurement positions are: supine (0°), lower seated (60°) and upper seated (68°). One of these allowed positions is reached, when the green OK-LED on the lifting mechanism indicator is glowing continuously. The liquefaction position is near the supine position and indicated by a blinking OK light.The MEG can be tilted by pressing the up/down-buttons located on the back of the device. For tilting the MEG downwards, the stirrup on the back has to be pushed down first.

**Please proceed as follows:**

A) tilting the MEG from liquefaction-position to measuring-position:

![](https://gitlab.com/obob/obob_ownft/uploads/716ee7411028f96aa942cdfa15e9e59a/DSC_0192.jpg)![](https://gitlab.com/obob/obob_ownft/uploads/f31cfa16399d5a1585cb5d6eb6f96ba2/DSC_0194.jpg)

press the UP-button continuously until the gantry stops automatically accompanied by a loud cracking noise – now the lower seated position (60°) is reached.

If you prefer the higher seated position, press the UP-button again continuously until the gantry stops at 68°.

![](https://gitlab.com/obob/obob_ownft/uploads/d7b03e78918c77653c50cc8922b92110/DSC_0195.jpg)

now the yellow LED (tension) is glowing – the gantry is still hanging on the rope of the linkage

press the DOWN-button for a short time until the green LED (ok) glows – the gantry is hanging stable on it's latches now

![](https://gitlab.com/obob/obob_ownft/uploads/942b3ed9bfab7974d5b7ca0392facddc/DSC_0196.jpg)

* **switch on the emergency call system (green button - next to the "nurse-symbol")******
* **switch on all necessary devices (Polhemus, open stimulation cabinett: green power-switch, blue vpixx-boxes, projector ...)******
* [**tune MEG-sensors (****MEG has to be adjusted in a measurement-position: supine (0°), 60° or 68° - doesn't work in liquefaction position: 25°!**](Sensor-tuning_27402431.html)**)******
* go to _Aquisition: control__→ Tools →__Tuner → File → Load tunings → Load other →_select latest tuning file_→ OK → Measure noise_
* in case sensors-bar(s) are missing: select missing sensor by pressing_Ctrl_and clicking_left mousebutton_at same time time (can also be done while noise-measuring is running)_→ Commands → Heat sensor →_sensor should come back → check by choosing _measure noise_(in case tuner not already running); to go back to all channels, click Ctrl and Shift at the same time while the mouse cursor is placed at a random position within the window showing the data of the single channel that had been selected
* in case a lot of sensors are missing, use the option: _Commands → Heat all sensors_ (this takes a while) → then select: _Measure noise_again
* in case noise level exceeds ~3 in average, and/or some sensors are very noisy (tall bars):_Tune_
* Now all sensors are getting tuned automatically. Wait until noise level is reduced to < 3 in average → then_Stop → Stop collector_
* _Then: File → Save tunings → Save other → select latest tuning and change filename to your initials and current date → OK → File → Exit_
* Now the MEG is ready for aquisitions
* **do an empty room measurement**Before starting the acquisition with participants, everybody is advised to do an empty room measurement for about 3 minutes with the MEG door closed!Project: empty_roomSubject: subject_subjectFile – Load settings: empty roomRecord raw --> omit HPI --> record MEG (3 min) --> save empty_room_68 (the number defines the gantry position in which MEG was recorded)Make sure that all sensors record data of good quality! In case you see a bad channel (though doing the tuning, heating, etc before), do heat the corresponding bad channel(s) again! This takes only few seconds! Restart the empty room measurement
