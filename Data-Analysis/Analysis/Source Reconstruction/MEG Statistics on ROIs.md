# MEG : How to run statistics on ROIs?

If you want to run statistical analyses on a specific set of regions and later plot the results you need to create ROI template_grids.

Please be aware, that if you have **2982 grid points** in the source data, you need to work here with a template_grid with 2982 grid points, otherwise the ROI and voxels will not match.

If your source data has **1457 grid points**(that means you reduced your data to grey matter only), the template_grid you work with here should also be reduced to 1457 grid points of grey matter only.

```actionscript3
load mni_grid_1_cm_2982pnts
template_grid = ft_convert_units(template_grid,'m');
% select only gray matter
atlas = ft_read_atlas(fullfile(worksp, '/obob/obob_ownft/external/fieldtrip/template/atlas/aal/ROI_MNI_V4.nii'));
atlas = ft_convert_units(atlas,'m');
 % assure that atlas and template_grid are expressed in the same units

template_grid.coordsys = 'mni'; 

%% this part only if you want 1457 voxels    
cfg = [];
cfg.atlas = atlas;
cfg.roi = atlas.tissuelabel;
mask = ft_volumelookup(cfg,template_grid);
template_grid.inside(template_grid.inside==1) = 0;
template_grid.inside(mask) = 1;


```

then we want to define ROIs based on the labels of grid points, which we created in [How can I get labels for my grid points?](27399640.html)

```actionscript3
%% load from "How can I get labels for my grid points"
load 'mni1457_gridlabels'; % if you have 1457 gridpoints
% or
load 'mni2982_gridlabels';% if you have 2982 gridpoints. you can also load the pre-computed one from here mnt/obob/staff/nsuess/code_snippets/mni2982_functionallabels

%% then we choose regions of interest based on the anatomical or functional names in the created file
chansel=ft_channelselection(cell({'Temporal*_L_*' 'Parietal*_L_*' 'Precentral*_L_*' 'Postcentral*_L_*'}), label);
[C,ia]=intersect(label, chansel);


idx_inside2all = find(template_grid.inside); %important to apply find the indeces of all fields not only inside...
ia_all = idx_inside2all(ia);


%% and then create a new grid
newgrid=template_grid;
newgrid.inside(template_grid.inside==1) = 0;
newgrid.inside(ia_all) = 1;
```

make sure to have selected the ROI of your interest via plotting the newgrid

```java
load('standard_mri');
cfg = [];
cfg.parameter = 'inside';

interp = ft_sourceinterpolate(cfg, newgrid, mri);

cfg = [];
cfg.funparameter  = 'inside';
ft_sourceplot(cfg, interp);
```

important to run the **statistics** on the newly chosen voxels →

```actionscript3
cfg.channel=data.label(ia);
```

and use the **newgrid for sourceplotting**
