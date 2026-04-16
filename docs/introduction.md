## Analysis workflow
A DarSIA analysis consists of setup, calibration and actual analysis. This notebook aims at giving a minimal introduction what are the main inputs required from the user and what commands are central in order to convert FluidFlower images to segmented maps (based on CO2 mass interpretation).

## Quick introduction to the terminal and how to interact with DarSIA
All the routines are fully defined as part of DarSIA. These scripts just envoke predefined workflows. The main user-input consists of prescribing the right metadata incl. paths, protocols etc. 

To run any scripts, the recommended way is to use the terminal. For this the example  commands in this file need to be copied and modified.

To run a given script with flags, and also to instruct the particular DarSIA, the main usage will consist of calling routines of the form:
```bash
python <scripts.py> --some-flags
```

## Create your own config files

Create your copy of the folder `config_example`, here called `my_config`. While these files are meant as template, you will need to both adapt the copied config files and also the paths in the following commands.

Note, that the config files are split into multiple config giles, separating common information and run-specific information. This separation is not stricklty needed, but recommended. The main changes needed to be performed are the location to the data, wehere results are stored etc. Some more details are provided in the following sections on setup, calibration and analysis.

## Setup

```bash
python scripts/setup.py --all --config my_config/single/common.toml my_config/run/run_050825.toml
```

If interested in intermediate results, add the flag `--show`.

Let's have a look at the required modifications of the config files. 
1. Open `my_config/run/run_050825.toml` to adapt run-specific metainformation in the `[data]` section:
- `folder`: adjust the path to the raw images
- `baseline`: identfy the main baseline image 
- `results`: set the path to your local folder for storing any results
2. Furthermore, we need to specify a bunch of protocols in the `[protocols]` section. Currently these are stored in the `data/protocol_050825.xlsx` file, it will contain multiple sheets that enable storing the following different information:
- `injection`: Defines the injection protocol, the first entry in the list identifies the path to the excel file, the second entry in the list identfies the sheet
- `imaging`: Defines the map between images and acquisition time
- `pressure_temperature`: Defines the experimental conditions
3. Open `my_config/single/commmon.toml` to adapt common metainformation in the `[depth]` section:
- `measurements`: Specify the path to the csv/excel file that collects point measurements of the depth of the rig
4. Continue with the `[labeling]` section:
- `colored_image`: Specify the path to the manually segmented image used for the identification of the different facies.
5. Continue with `[facies]` section:
- `props`: Specify the path to the excel sheet with info on the different facies, like porosity.
- `id`: List the relevant id's of the segmentation (may not be clear a prior - you may want to run the setup with `--show` flag first and get an idea of what id's are available).
- `[facies.N]`: Link facies `N` (see excel sheet representation of lab measurements) with label `id` (DarSIA interpretation of the segmentation). For this, you need to modify the `labels` list and list all relevant `id` values.

## Calibration of pH indicator specific color paths
Next we can run the calibration script, which identifies run-specific characteristic colors in a small set of calibration images.

The aim will be to run the command:

```bash
python scripts/calibration.py --color-paths --config my_config/single/common.toml my_config/run/run_050825.toml
``` 

Again, we will need to pay attention to the config files.

Open once again `my_config/run/run_050825.toml`, and check out the `[color_paths]` section. The most important entries here are (it is not expected that the options need modifications):
1. `baseline_images`: Pick at least one or more images. More allow for better understanding of typical fluctuations in the baseline colors due to fluctuating illumination.
2. `reference_label`: Pick the label you would like to use for picking a characteristic color mpa in later plots. This is not a crucial step for the analysis. To pick a value, it helps to understand DarSIA's segmentation of the geometry into sand layers here.
3. `[color_paths.data.image_time_interval.TEXT]`: Specify an interval of calibration data by defining:
- `start`: Define the beginning of the calibration inverval in the format HH:MM:SS
- `end`: Define the end of the calibration interval in the format HH:MM:SS
- `num`: Define the number of equidistant times considered for calibration
- `tol`: Define a tolerance which will be allowed if images with exact desired acquisition time are not available. A close one will be chosen then within the provided tolerance. Use the format HH:MM:SS.

## Calibration of color-based CO2 mass analysis

Next, we run the calibration of the mass analysis, i.e., the conversion of colors to CO2 mass.

```bash
python scripts/calibration.py --mass --config my_config/single/common.toml my_config/run/run_050825.toml
``` 
