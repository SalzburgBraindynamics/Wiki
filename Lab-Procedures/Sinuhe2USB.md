# MEG : How to copy data from Sinuhe (=Acquisition-PC) to USB-Stick

Sometimes just copying the .fif files doesn't work (the files are corrupted), so the best way to copy the files is using the linux rsync in the following wayrsync -avhW --no-compress  --progress SRC DESTFor example, go to the data foldercd /neuro/data/sinuhe/my_own_project/160124and thenrsync -avhW --no-compress  --progress *.fif /media/MYUSBSTICK/MYFOLDER
