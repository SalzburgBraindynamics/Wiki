# MEG : How can I get labels for my grid points?

This code creates labels of grid points. Each grid point will receive an anatomical (or functional) label or "NA" if no such label is offered by the atlas you chose.

You need this if you want to create regions of interest based on anatomical locations.

Choose the atlas accordingly (anatomically/functionally).

```actionscript3
worksp='/mnt'
cd /mnt/obob/obob_ownft/external/fieldtrip/private

% choose the atlas and the template_grid for making the labels for a given grid (depending on the atlas this might be anatomical or functional labels)
atlas=ft_read_atlas(fullfile(worksp, '/obob/obob_ownft/external/fieldtrip/template/atlas/aal/ROI_MNI_V4.nii')); % atlas with anatomical labels
load mni_grid_1_cm_2982pnts.mat


% assure that atlas and template_grid are expressed in the same units
atlas = ft_convert_units(atlas,'m');
template_grid = ft_convert_units(template_grid,'m');


% in this example only the grey matter is used, if you want to label the whole brain, i.e. if you have 2982 voxels, skip this step (following 7 lines)
cfg = [];
cfg.atlas = atlas;
cfg.roi = atlas.tissuelabel;
cfg.inputcoord = 'mni';
mask = ft_volumelookup(cfg,template_grid);
template_grid.inside(template_grid.inside==1) = 0;
template_grid.inside(mask) = 1;

%%
elec = struct;
elec.pos(:,:) = template_grid.pos(template_grid.inside,:);
clear template_grid

%% create for 1457 grey matter voxels the corresponding label
template_grid=elec;
for ii=1:length(template_grid.pos)
  tmplabel = atlas_lookup(atlas, template_grid.pos(ii,:),'coordsys', 'mni')
  if isempty(tmplabel)
    label(ii)={['None_' num2str(ii)]};
  else
    label(ii)={[cell2mat(tmplabel(1)) '_' num2str(ii)]};
  end
  ii
end

%%
save /mni1457_gridlabels label

% or if you have 2982 grid points
save /mni2982_gridlabels label

 
```
