# SimTO Dataset

This repository hosts the **SimTO Dataset**, a large-scale collection of **2,071 bespoke, manufacturable soft gripper designs**. The designs were produced using [**SimTO**](https://kurtenkera.github.io/SimTO/) - a simulation-based topology optimization framework that generates highly specialized soft robotic grippers for *feature-rich* objects (e.g., gears with sharp tooth profiles or corals with fragile protrusions) that challenge existing robotic grippers due to their high topological variability.

<br>
<p align="center">
  <img src="SimTO_framework.png" alt="SimTO Framework Diagram" width="780">
  <br>
  <strong>The SimTO framework:</strong> <em> Given an arbitrary feature-rich object, SimTO generates bespoke soft fingers which conform to that object's shape under actuation.</em>
</p>
<br>

This dataset is intended for researchers and engineers in the field of soft robotics, particularly those focused on gripper design and optimization.

## Directory Structure

The dataset includes designs for five different feature-rich objects: a curvy ball, gear, hourglass, spiky ball and star. For each object, **64 independent optimization runs** were performed by varying three key design hyperparameters: $E_o$ (object Young's modulus), $E_g$ (gripper Young's modulus) and $v_f$ (volume fraction). These runs are reflected in the dataset's directory structure, with each hyperparameter's value encoded in the folder names as `objYM`, `fingYM` and `vf` respectively. Below is a high-level view of the dataset structure:

```
SimTO-Dataset/
├── curvyball/
│   ├── curvyball_objYM=0.46MPa_fingYM=0.46MPa_vf=0.20/  <-- Sample data folder
│   │   ├── curvyball_run01.zip  <-- Compressed folder containing optimization results
│   │   │   ├── cps2maxforces_3D_D1.npy  <-- 3D contact points and force data (numpy arrays)
│   │   │   ├── cps2maxforces_2D_D1.npy  <-- 2D contact points and force data (numpy arrays)
│   │   │   ├── D1_1D_binary.npy  <-- 1D binary data (numpy array)
│   │   │   ├── D1_3D_binary.npy  <-- 3D binary data (numpy array)
│   │   │   ├── D1_2D_Forces.png  <-- 2D contact forces visualization
│   │   │   ├── D1_iteration_100.png  <-- Topology optimization iteration screenshot
│   │   │   ├── D1_leftfinger.msh  <-- Left gripper mesh (for simulation)
│   │   │   └── D1_leftfinger.stl  <-- Left gripper STL file (for 3D printing)
│   ├── curvyball_objYM=0.46MPa_fingYM=0.46MPa_vf=0.25/
│   └── ... (up to 64 sub-directories per target object, each containing a zip archive)
├── gear/
├── hourglass/
├── spikyball/
└── star/
```

## File Breakdown

The following 8 file types are found in each of the subdirectories: 

### 1. Contact Force Distributions used for SimTO design optimization
* **`cps2maxforces_3D_Di.npy`**: A Python dictionary mapping $\tilde{N}_f$ 3D contact points on the left soft finger to 3D contact forces for the $i$ th simulated design.
* **`cps2maxforces_2D_Di.npy`**: A Python dictionary mapping $N_f \leq \tilde{N}_f$ 2D contact points on the left soft finger to 3D contact forces for the $i$ th simulated design.
* **`Di_2D_Forces.png`**: A visualization of the `cps2maxforces_2D` forces plotted on their associated contact points for the $i$ th design.

### 2. Design Representations
* **`Di_1D_binary.npy`**: A $(10500,)$ numpy array of 1s and 0s representing the final optimized design (reshapable to $150 \times 70$).
* **`Di_iteration_j.png`**: A 2D plot of the `Di_1D_binary.npy` at iteration $j$.
* **`Di_3D_binary.npy`**: A $(150, 70, 30)$ numpy array of 1s and 0s representing the optimized design extruded with a thickness of 30mm.

### 3. Geometry & Fabrication
* **`Di_leftfinger.msh`**: Volumetric mesh of the $i$ th soft finger used for simulation in [**Taccel**](https://taccel-simulator.github.io/).
* **`Di_leftfinger.stl`**: STL file of the voxelized volumetric mesh, ready for 3D printing and physical validation.

## Inspecting the Data in Python

You can use the following snippet to load a design and visualize its structure:

```python
import numpy as np
import matplotlib.pyplot as plt

# Path to the sample design
file_path = 'object_01/design_01/D1_1D_binary.npy'

# Load the binary design
design = np.load(file_path)

# Reshape to 2D (150 rows, 70 columns)
# Note: Ensure the reshape dimensions match your optimization grid
design_2d = design.reshape(150, 70)

# Visualization
plt.figure(figsize=(6, 10))
plt.imshow(design_2d, cmap='gray_r')
plt.title('SimTO Optimized Design (Binary)')
plt.xlabel('Width')
plt.ylabel('Height')
plt.show()
```

The following snippet can be used to plot `cps2maxforces_2D` or `cps2maxforces_2D`  on a given volumetric mesh (.msh):

```python
def plot_mesh_and_CPs(self, title: str, CPs: np.ndarray, forces: np.ndarray, 
                        mesh_filepath: str, save_filepath: str = None):

    pv_mesh = pv.read(mesh_filepath)

    pl = pv.Plotter(off_screen=True)  # Off-screen rendering needed for saving PNG
    pl.add_axes()
    pl.add_title(title, font_size=20)
    pl.add_mesh(pv_mesh, color='green', opacity=0.2, show_edges=True, 
                edge_color='black', line_width=0.5)
    pl.add_points(CPs, color='red', point_size=10)
    pl.add_arrows(CPs, forces, mag=self.animation_force_mag, show_scalar_bar=False, color='purple')


    if save_filepath:
        pl.screenshot(save_filepath)  # Save as PNG
    else:
        pl.show()
    return
```