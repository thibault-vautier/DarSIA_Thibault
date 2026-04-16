# Data and Image Analysis
Once calibrated, DarSIA can assist in various data analysis procedures. This list shows a complete overview of the general capabilities. Custom routines may be required to answer specific questions based on interpretations of the images, e.g., CO2 mass map or gas/aqueous segmented phases.

## The main run script
All analysis steps in this overview are controlled via the script `scripts/analysis.py` and different flags control which analysis routine is envoked.

## Config files
Again, the routines will require a config file to steer the analysis step in addition to common items already used during setup and calibration. In this overview, we will use an additional config file for the analysis steps, here referred to as `analysis.toml`, cleanly separated from the common and run-specific config files. This allows to again reuse the same analysis config file across different runs.

A central item for all steps is the `analysis.toml` file are the `[analysis.data.image_time_interval.TEXT]` sections, used to define the actual data to be analysed. This section is used across all presented analysis steps.

## Cropping raw images
For presentation purposes it is helpful to just geometrically process the raw images into the main ROI. This step is here referred to as 'cropping'. Use the flag `--cropping` to invoke cropping of the chosen images (see above):

```bash
python scripts/analysis.py --cropping --config config_example/single/common.toml config_example/run/run_XYZ.toml config_example/single/analysis.toml
```

The cropped images in `jpg` format can be then found in the results folder under the `cropped_images` subfolder. (Reminder: Check the `common.toml` config for the main `results` folder within the `[data]` section.)

## Mass 
The calibrated mass analysis can be applied to batches of images and convert images to spatial mass maps. These will be stored as `npz` files in your main `results` directory. These files cannot be investigated visually but provide the basis for further analysis. Note that the step of converting an image to its mass interpretation is costly and thus should be considered to be performed only once for multiple analysis steps (detailed in the notebook on post-analysis).

```bash
python scripts/analysis.py --mass --config config_example/single/common.toml config_example/run/run_XYZ.toml config_example/single/analysis.toml
```

It is possible to restrict the mass analysis to regions of interest (ROI). For this, in the config file, one can add as many sections `[analysis.mass.roi.TEXT]` with separate user-defined identifiers `TEXT` of the form:

```toml
[analysis.mass.roi.box2]
name = "Box 2"
corner_1 = [1.45, 0.0]
corner_2 = [2.745, 1.5]
```

In addition to creating `npz` files of the spatial CO2 mass, this analysis step also performs an integration of the CO2 density over the different boxes. If none is provided, the full geometry will be considered. The final integrated values are stored in a separate `csv` file under `sparse_data/integrated_mass.csv` within the main results directory, ready for further analysis or plotting.

## Segmentation
Segmentation of phases using contour lines following the interpreted saturation values of the gaseous phase and the concentration values of the aqueous phase allows for visualization of the multiphase CO2-water mixture. For this the flag `--segmentation` needs to be used, together with the config files.

```bash
python scripts/analysis.py --segmentation --config config_example/single/common.toml config_example/run/run_XYZ.toml config_example/single/analysis.toml
```

The segmentation is configured within `analysis.toml`. E.g., in order to trigger contour lines associated to the aqueous phase, add this block. Here the `mode` is essential to address the right concentration values. The `thresholds` identify the values of the contour lines. The `color` in RGB format defines the color of the contour lines, while `alpha` allows to use a grading in the colors, according to the different threshold values. Finally, `linewidth` controls the displayed line.

```toml
[analysis.segmentation.aqueous]
label = "CO2(aq)"
mode = "concentration_aq"
thresholds = [0.01, 0.1, 0.5, 0.9]
color = [0, 255, 0]
alpha = [0.5, 0.7, 0.9, 1.0]
linewidth = 8
```

In order to add contour lines associated to the saturation of the gaseous phase, use `saturation_g` for the `mode`. The remaining values are chosen to control the final contour lines with identical interpretation as the values above.

```toml
[analysis.segmentation.gas]
label = "CO2(g)"
mode = "saturation_g"
thresholds = [0.01, 0.1, 0.5, 0.9]
color = [255, 0, 0]
alpha = [0.3, 0.5, 0.7, 0.9]
linewidth = 5
```

The final segmented files are then stored in `jpg` format in a subfolder `segmentation` of the main results directory.