# Selection of images

Selecting images is necessary for various tasks, from calibration to analysis. There exist various ways to enable picking images through the config. Either by direct selection of paths, image times, or time intervals with equidistant time steps. This document presents the three different ways.

## Option 1: Time intervals

Define image selection using a time interval with equidistant time steps. This method automatically generates a list of target times within a specified interval, then finds the closest images in your dataset for each target time.

**Configuration:**
```toml
[color_paths.data.image_time_interval.calibration1]
start = "00:00:00"
end = "02:30:00"
num = 10
tol = "00:10:00"
```

**Parameters:**
- `start` (string, required): Start time in HH:MM:SS format
- `end` (string, required): End time in HH:MM:SS format
- `num` (integer, required): Number of equidistant images to extract within the interval
- `tol` (string, optional): Tolerance in HH:MM:SS format for matching images (images outside this tolerance are discarded)

**How it works:**
1. A set of `num` target times are generated uniformly distributed between `start` and `end`
2. For each target time, the script searches for the closest image in your dataset
3. If `tol` is specified, only images within the tolerance window are selected
4. The selected images are used for calibration

**Example:**
In the example above, 10 images are selected uniformly between 00:00:00 and 02:30:00 (15 minute intervals). The tolerance of ±00:10:00 means that images must be within 10 minutes of the target time to be included.

## Option 2: Specific image times

Select images at specific times without automatic distribution. This method is useful when you want to choose particular moments in the experiment for calibration or analysis.

**Configuration:**
```toml
[color_paths.data.image_times.calibration2]
times = ["01:30:00", "02:00:00", "02:30:00"]
tol = "00:10:00"
```

**Parameters:**
- `times` (array of strings, required): List of target times in HH:MM:SS format
- `tol` (string, optional): Tolerance in HH:MM:SS format for matching images (images outside this tolerance are discarded)

**How it works:**
1. A list of target times is explicitly specified
2. For each target time, the script searches for the closest image in your dataset
3. If `tol` is specified, only images within the tolerance window are selected
4. The selected images are used for calibration or analysis

**Example:**
In the example above, 3 images are selected at 01:30:00, 02:00:00, and 02:30:00. The tolerance of ±00:10:00 means that images must be within 10 minutes of each target time to be included.

**Use cases:**
- Selecting specific time points when you know the experiment has important events
- Manually choosing calibration points at irregular intervals
- Targeting images at specific experimental phases or conditions

## Option 3: Direct file paths

Select images by directly specifying file paths. This method provides the most control when you want to use specific images regardless of their timestamps.

**Configuration:**
```toml
[color_paths.data.image_paths.calibration3]
paths = ["DSC04574.JPG", "DSC04640.JPG", "DSC05000.JPG", "DSC0600*.JPG"]
```

**Parameters:**
- `paths` (array of strings, required): List of image filenames, relative paths, or glob patterns. Supports wildcards:
  - `"*.JPG"` - matches all JPG files
  - `"DSC0600*.JPG"` - matches DSC06000.JPG, DSC06001.JPG, ..., DSC06009.JPG, etc.
  - `"DSC04574.JPG"` - matches a specific file

**How it works:**
1. Image paths or glob patterns are directly specified in the configuration
2. Glob patterns are expanded to match all files in the data folder
3. The specified or matched files are located in the data folder
4. The selected images are used for calibration or analysis

**Example:**
In the example above, specific JPG files are selected by name (`DSC04574.JPG`, `DSC04640.JPG`, `DSC05000.JPG`) and a glob pattern (`DSC0600*.JPG`) that matches all files starting with `DSC0600`. The paths are relative to the data folder specified in the `[data]` section.

**Glob pattern examples:**
- `"*.JPG"` - Select all JPG images
- `"DSC0*.JPG"` - Select all images starting with `DSC0`
- `"DSC0600?.JPG"` - Select images DSC06000.JPG through DSC06009.JPG (single character wildcard)
- `"[DSC04,DSC05]*.JPG"` - Select images starting with either `DSC04` or `DSC05`

**Use cases:**
- When you have pre-selected images from external sources
- For reproducible results when specific images are critical
- When timestamps are unreliable or unavailable
- Combining images from different experiments or runs

