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

The dataset includes designs for five different feature-rich objects: a curvy ball, gear, hourglass, spiky ball and star. For each object, **64 independent optimization runs** were performed by varying three key design hyperparameters: $E_o$ (object Young's modulus), $E_g$ (gripper Young's modulus) and $v_f$ (volume fraction). Data from each optimization run is saved in a directory that encodes the specific hyperparameter values as `objYM`, `fingYM` and `vf` respectively. Below is a high-level view of the dataset structure:

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

The following 8 file types are found in each of the sub-directories: 

### Contact Force Distributions used for SimTO Design Optimization
* **`cps2maxforces_3D_Di.npy`**: A Python dictionary mapping $\tilde{N}_f$ 3D contact points on the left soft finger to 3D contact forces for the $i$ th simulated design.
* **`cps2maxforces_2D_Di.npy`**: A Python dictionary mapping $N_f \leq \tilde{N}_f$ 2D contact points on the left soft finger to 2D contact forces for the $i$ th simulated design.
* **`Di_2D_Forces.png`**: A visualization of the contact forces from `cps2maxforces_2D_Di.npy` for the $i$ th design.

### Design Representations
* **`Di_1D_binary.npy`**: A $(70*150,)$ numpy array of 1s and 0s representing the $i$ th design.
* **`Di_iteration_j.png`**: A 2D plot of `Di_1D_binary.npy` at iteration `j` of simulation-based topology optimization.
* **`Di_3D_binary.npy`**: A $(70, 150, 30)$ numpy array of 1s and 0s representing the $i$ th design extruded with a thickness of 30mm.

### Geometry & Fabrication
* **`Di_leftfinger.msh`**: A volumetric mesh of the $i$ th soft finger used for simulation in [**Taccel**](https://taccel-simulator.github.io/).
* **`Di_leftfinger.stl`**: STL file of the voxelized volumetric mesh, ready for 3D printing and physical validation.

## Inspecting the Data in Python

You can use the following snippet to load a design and visualize its 2D structure:

```python
import numpy as np
import matplotlib.pyplot as plt

# Path to the sample design
file_path = 'object_01/run_01/D1_1D_binary.npy'

# Load the binary design
design = np.load(file_path)

# Reshape to 2D 70mm x 150mm design
design_2d = design.reshape((70,150), order = 'F')

# Visualization
plt.figure(figsize=(6, 10))
plt.imshow(-design_2d, cmap='gray', interpolation='none')
plt.title('SimTO Optimized Design (Binary)')
plt.show()
```

The following snippet can be used to plot the contact forces from `cps2maxforces_2D` or `cps2maxforces_3D` on a given volumetric mesh:

```python
import pyvista as pv
import numpy as np

# Loading cps2maxforces
cps2maxforces_filepath = "object_01/run_01/cps2maxforces_3D_D2.npy"
cps2maxforces = np.load(cps2maxforces_filepath, allow_pickle=True).item()

# Defining CPs and forces from the Python dictionary
CPs = np.array(list(cps2maxforces.keys()))
forces = np.array(list(cps2maxforces.values()))
mesh_filepath = "object_01/run_01/D2_leftfinger.msh"

# Plotting forces using Pyvista
pv_mesh = pv.read(mesh_filepath)
pl = pv.Plotter(off_screen=False) 
pl.add_axes()
pl.add_title("Force Visualization", font_size=10)
pl.add_mesh(pv_mesh, color='green', opacity=0.2, show_edges=True, 
            edge_color='black', line_width=0.5)
pl.add_points(CPs, color='red', point_size=10)
pl.add_arrows(CPs, forces, mag=0.005, show_scalar_bar=False, color='purple')
pl.show()
```