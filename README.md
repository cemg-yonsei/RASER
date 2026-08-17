# RASER (Recursive Aggregate Segmentation by Erosion and Reconstitution)

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Language: MATLAB](https://img.shields.io/badge/Language-MATLAB-blue.svg)
![Version](https://img.shields.io/badge/version-v1.0-green.svg)

## 📌 Overview

**RASER** is an open-source MATLAB software designed to objectively segment individual particles in densely packed materials. It employs a **dual-loop architecture** (recursive erosion and iterative reconstitution) to resolve connectivity conflicts without subjective manual parameter tuning.

---

## 🔬 How RASER Works

RASER starts from a binary image in which the particle phase has already been identified. The segmentation proceeds as follows:

1. **Connected-component identification**  
   Initial particles and fused clusters are identified according to the user-defined voxel connectivity.

2. **Size-based classification**  
   Detected objects are divided into **coarse** and **small** particles using the threshold `d_min`. Small particles are retained without further erosion, while coarse particles are passed to the erosion–reconstitution process.

3. **Surface erosion**  
   At the current erosion depth, surface voxels are removed layer by layer. The indices of all removed voxels are stored so that the original particle volume can later be reconstructed.

4. **Particle-core identification**  
   After erosion, newly separated connected components are identified as individual particle cores.

5. **Surface reconstitution**  
   Removed voxel layers are restored in reverse order (Last-In, First-Out). Voxels are reassigned to neighboring particle cores according to the selected connectivity.

6. **Interface conflict resolution**  
   If a removed voxel is adjacent to multiple particles, RASER resolves the competing assignments deterministically to maintain unique particle labels.

7. **Iterative convergence**  
   Steps 2–6 are repeated at the same erosion depth until the number of identified particles no longer changes.

8. **Recursive convergence**  
   The erosion depth is then increased by one voxel and the iterative process is repeated. RASER terminates when increasing the erosion depth produces no additional particle separation.

## 📁 Repository Structure & Function Reference

The source code is modularized into specialized functions to ensure transparency and reproducibility.

```text
RASER/
├── src/
│   ├── RASER_Segmentation_Main.m           # Main execution script
│   ├── imread_big.m                        # Handles >4GB TIFF stacks
│   ├── discretize_large_particle.m         # Size-based classification (d_min)
│   ├── erode_surface.m                     # Recursive erosion logic
│   ├── reconstitute_surface_particle.m     # Iterative volume restoration
│   ├── findNeighbours_voxels.m             # Neighborhood search
│   ├── remove_overlap.m                    # Interface conflict resolution
│   ├── find_interface.m                    # Particle boundary handling
│   ├── img_output.m                        # Binary TIFF export
│   ├── img_output_label.m                  # Labeled TIFF export
│   └── plot_diameter.m                     # Size distribution analysis
└── data/                                   # Input/Output directory
```

### 🛠️ Core Function Descriptions

#### `imread_big`
Loads large (>4 GB) TIFF stacks into a 3D array (`uint8`, `uint16`, or `uint32`).

#### `discretize_large_particle`
Classifies particles into:

- **Coarse** particles (processed through the recursive segmentation loop)
- **Small** particles (excluded from recursive processing)

based on the user-defined threshold `d_min`.

#### `erode_surface`
Performs recursive erosion. Surface voxels removed during each erosion stage are indexed and stored for later reconstruction.

#### `reconstitute_surface_particle`
Core reconstruction routine that restores eroded particle volume. A Last-In, First-Out (LIFO) strategy is used to reassign removed voxels to the nearest particle core while preserving total particle volume.

#### `findNeighbours_voxels`
Identifies neighboring voxels according to the selected connectivity scheme.

#### `remove_overlap`
Removes geometric overlaps occurring at interfaces between neighboring particles.

#### `find_interface`
Resolves topological conflicts when multiple particles attempt to claim the same voxel, ensuring objective voxel ownership assignment.

#### `img_output_label`
Generates a labeled volume (`labelmatrix`) compatible with Fiji/ImageJ and other 3D visualization software.

---

## ⚙️ Prerequisites & Data Requirements

### Environment

- MATLAB R2024b or later
- Image Processing Toolbox

### Input Data Format

- Uncompressed TIFF stack
- Binary image only:
  - Background = `0`
  - Particle = `255`
- Data type must be `uint8`
- For 3D analysis, the image stack must contain at least **3 slices (pages)**

---

## 🚀 Usage Instructions

### 1. Setup

Place the binary TIFF image in:

```text
data/Image_source/
```

Example:

```text
binary_20_20_20_July_small.tif
```

### 2. Parameter Configuration (`RASER_Segmentation_Main.m`)

#### `filename`

Input TIFF filename.

Example:

```matlab
filename = 'binary_20_20_20_July_small.tif';
```

#### `conn`

Voxel connectivity definition.

**2D connectivity**

- `4`  = edge-connected neighbors
- `8`  = edge- and corner-connected neighbors

**3D connectivity**

- `6`  = edge-connected neighbors
- `18`  = face- and edge-connected neighbors
- `26`  = face-, edge-, and corner-connected neighbors

#### `d_min`

Minimum characteristic particle-size threshold used for computational filtering.

RASER classifies detected particles into:

- **Coarse particles**: passed to the iterative erosion–reconstitution process.
- **Small particles**: retained without further erosion.

For 3D images, the characteristic size is defined using the intermediate principal-axis length obtained from `regionprops3`. For 2D images, the minor-axis length obtained from `regionprops` is used.

`d_min` primarily controls which particles require further processing and does **not** determine the erosion depth used for segmentation. The erosion depth is automatically determined by the recursive convergence procedure.

Setting

```matlab
d_min = 1;
```

for exhaustive segmentation of all detected particles.

### 3. Execution

Run:

```matlab
RASER_Segmentation_Main
```

The algorithm automatically performs:

1. Particle classification
2. Recursive erosion
3. Iterative reconstitution
4. Conflict resolution
5. Label generation

Final outputs will be saved in:

```text
data/Output_image/
```

---

## 🖼️ Example Result

RASER successfully separates artificially fused touching aggregates while preserving their original morphology.

Example dataset:

```text
binary_20_20_20_July_small.tif
```

For 3D visualization with unique colors assigned to individual particles:

1. Open the labeled TIFF in Fiji/ImageJ.
2. Install the MorphoLibJ plugin.
3. Use:

```text
Label Images → Label to RGB
```

to generate colorized particle renderings.

---

## 📖 Citation

If you use RASER in your research, please cite:

> Haneul Lee, D.F. Tiana Razakamandimby R., Kyoungsoo Park,
> "RASER: An open-source software for particle segmentation in densely packed X-ray CT images",
> SoftwareX (Under Review).

---

## 📄 License

This project is licensed under the MIT License.
