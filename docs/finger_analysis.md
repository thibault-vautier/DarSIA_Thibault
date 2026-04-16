# Finger Analysis Configuration

Finger analysis detects and analyzes fingering phenomena in multiphase flow experiments. Fingering occurs when a denser fluid displaces a lighter fluid, creating instabilities that produce finger-like patterns at the interface. This document explains how to configure finger analysis in DarSIA.

## Overview

Finger analysis requires two main configuration components:

1. **Analysis Mode and Threshold** - defines what to analyze and sensitivity
2. **Regions of Interest (ROIs)** - defines spatial regions where fingers are detected

## Configuration Structure

```toml
[analysis.fingers]
mode = "concentration_aq"
threshold = 0.01

[analysis.fingers.roi.top]
name = "Close to Gas-Liquid Interface"
corner_1 = [0.0, 1.05]
corner_2 = [2.745, 0.95]

[analysis.fingers.roi.storage]
name = "Storage Region"
corner_1 = [0.0, 1.05]
corner_2 = [2.745, 0.3]
```

## Main Configuration

### `[analysis.fingers]` Section

**Parameters:**
- `mode` (string, required): The physical quantity to analyze for detecting fingers
  - `"concentration_aq"` - Aqueous phase CO2 concentration
  - `"saturation_g"` - Gaseous phase saturation
- `threshold` (float, required): Threshold value above which pixels are considered part of a finger
  - Range typically between 0.0 and 1.0
  - Lower values detect more subtle fingering
  - Higher values only detect pronounced fingers

**How it works:**
1. The selected physical quantity (concentration or saturation) is extracted from the analyzed images
2. Pixels with values exceeding the threshold are identified as potential fingers
3. Connected regions are grouped and analyzed as individual fingers
4. Finger characteristics (area, length, location) are computed

**Example:**
```toml
[analysis.fingers]
mode = "concentration_aq"
threshold = 0.01
```

This configuration analyzes aqueous CO2 concentration and identifies regions where concentration exceeds 0.01 (1%) as fingers.

## Regions of Interest (ROIs)

### Purpose

ROIs allow you to restrict finger analysis to specific spatial regions of interest. Multiple ROIs can be defined with unique identifiers.

### Configuration

For each ROI, define a subsection under `[analysis.fingers.roi.TEXT]` where `TEXT` is your custom identifier.

**Parameters:**
- `name` (string, required): Descriptive name for the ROI (used in output labels)
- `corner_1` (array of 2 floats, required): Coordinates of the first corner `[x, y]` in meters
- `corner_2` (array of 2 floats, required): Coordinates of the opposite corner `[x, y]` in meters

**Coordinate System:**
- Origin (0, 0) is typically at the bottom-left of the domain
- X increases to the right (width direction, 0 to ~2.745 m)
- Y increases upward (height direction, 0 to ~1.5 m)
- Corners can be specified in any order (automatic normalization)

### Examples

**Example 1: Top ROI (Interface Region)**
```toml
[analysis.fingers.roi.top]
name = "Close to Gas-Liquid Interface"
corner_1 = [0.0, 1.05]
corner_2 = [2.745, 0.95]
```

This defines a thin horizontal band near the gas-liquid interface:
- X range: 0.0 to 2.745 m (full width)
- Y range: 0.95 to 1.05 m (±0.05 m around 1.0 m height)

**Example 2: Storage Region (Lower Domain)**
```toml
[analysis.fingers.roi.storage]
name = "Storage Region"
corner_1 = [0.0, 1.05]
corner_2 = [2.745, 0.3]
```

This defines a large lower region:
- X range: 0.0 to 2.745 m (full width)
- Y range: 0.3 to 1.05 m (lower 75% of domain)

**Example 3: Left Half Only**
```toml
[analysis.fingers.roi.left]
name = "Left Half"
corner_1 = [0.0, 0.0]
corner_2 = [1.3725, 1.5]
```

This restricts analysis to the left half of the domain.

## Common Use Cases

### 1. Interface Fingering Detection
Focus on fingers developing at the gas-liquid interface:
```toml
[analysis.fingers]
mode = "concentration_aq"
threshold = 0.01

[analysis.fingers.roi.interface]
name = "Interface Zone"
corner_1 = [0.0, 0.95]
corner_2 = [2.745, 1.05]
```

### 2. Bottom Instability Analysis
Detect fingers near the bottom boundary:
```toml
[analysis.fingers]
mode = "saturation_g"
threshold = 0.05

[analysis.fingers.roi.bottom]
name = "Bottom Boundary Layer"
corner_1 = [0.0, 0.0]
corner_2 = [2.745, 0.2]
```

### 3. Multi-Region Fingering Comparison
Compare fingering in different regions:
```toml
[analysis.fingers]
mode = "concentration_aq"
threshold = 0.01

[analysis.fingers.roi.upper]
name = "Upper Region"
corner_1 = [0.0, 0.75]
corner_2 = [2.745, 1.5]

[analysis.fingers.roi.lower]
name = "Lower Region"
corner_1 = [0.0, 0.0]
corner_2 = [2.745, 0.75]
```

## Output and Results

Finger analysis produces:
- **Visual output**: Images with fingers highlighted and labeled
- **Statistical data**: Finger counts, sizes, locations for each ROI and time point
- **Temporal data**: How finger characteristics evolve over time

Results are typically stored in the results folder with subdirectories organized by ROI and analysis type.

## Tips and Best Practices

1. **Threshold Selection**
   - Start with a moderate threshold (e.g., 0.01-0.05) and adjust based on visual inspection
   - Lower thresholds capture smaller fingers but may detect noise
   - Higher thresholds detect only pronounced fingers

2. **ROI Sizing**
   - Make ROIs large enough to contain meaningful fingers
   - Very thin ROIs may miss fingers passing through
   - Overlapping ROIs are allowed and useful for comparison

3. **Multiple Configurations**
   - Define multiple finger analysis sections with different modes/thresholds to compare results
   - Use different ROIs to isolate specific phenomena

4. **Coordinate Verification**
   - Verify coordinates match your domain geometry
   - Use visual inspection to confirm ROI placement
