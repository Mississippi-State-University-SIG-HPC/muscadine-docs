# Using Fierro 

## Overview

Fierro is a software made by Los Alamos National Laboratory and is a multiphysics code for compressible material dynamics and quasi-static solid mechanics. On Muscadine, the multiphysics solver is built with the HIP backend, enabling GPU-accelerated execution via MATAR/Kokkos.

---

## Fierro Input File Reference

To run a simulation, Fierro requires a YAML input file that defines the physics, materials, and geometry. The Fierro input file is organized into several key sections, each controlling a specific aspect of the simulation.

<details>
  
  <summary><code>dynamic_options</code> : Time integration parameters</summary>
  
  ---
  
  Controls time integration parameters for the simulation.
  
  | Field | Type | Description |
  |---|---|---|
  | `time_initial` | double | Start time for the simulation |
  | `time_final` | double | End time for the simulation |
  | `dt_min` | double | Minimum allowable time step |
  | `dt_max` | double | Maximum allowable time step |
  | `dt_start` | double | Initial time step |
  | `dt_cfl` | double | CFL (Courant-Friedrichs-Lewy) condition factor |
  | `cycle_stop` | integer | Maximum number of computational cycles/iterations |
  | `fuzz` | double | Small epsilon value for floating point comparisons |
  | `tiny` | double | Very small value threshold |
  | `small` | double | Small value threshold |
  | `rk_num_stages` | integer | Number of Runge-Kutta stages for time integration |
  
  ---
  
</details>
  
<details>
  
  <summary><code>mesh_options</code> : Computational mesh definition</summary>
  
  ---
  
  Defines the computational mesh. You can either generate a mesh internally or read one from a file.
  
  | Field | Type | Description |
  |---|---|---|
  | `num_dims` | integer | Number of dimensions (typically 2 or 3) |
  | `source` | enum | Mesh source: `file` or `generate` |
  | `file_path` | string | Path to mesh file (when `source: file`) |
  | `type` | enum | Mesh type to generate: `box`, `polar` |
  | `origin` | `[x, y, z]` | Origin coordinates |
  | `length` | `[lx, ly, lz]` | Length in each dimension |
  | `num_elems` | `[nx, ny, nz]` | Number of elements per dimension |
  | `polynomial_order` | integer | Order of basis functions |
  
  **Polar mesh only:**
  
  | Field | Type | Description |
  |---|---|---|
  | `inner_radius` | double | Inner radius |
  | `outer_radius` | double | Outer radius |
  | `starting_angle` | double | Starting angle |
  | `ending_angle` | double | Ending angle |
  | `num_radial_elems` | integer | Number of radial elements |
  | `num_angular_elems` | integer | Number of angular elements |
  
  **Scaling:**
  
  | Field | Type | Description |
  |---|---|---|
  | `scale_x` | double | X-axis scale factor |
  | `scale_y` | double | Y-axis scale factor |
  | `scale_z` | double | Z-axis scale factor |
  
  ---
  
</details>
  
<details>
  <summary><code>output_options</code> : Output frequency, format, and fields</summary>
  
  ---
  
  Manages simulation output frequency, format, and which fields are written.
  
  | Field | Type | Description |
  |---|---|---|
  | `timer_output_level` | string | Verbosity of timing information |
  | `output_file_format` | string | Output format, e.g. `viz`, `viz_and_state` |
  | `graphics_time_step` | double | Simulation time interval between output dumps |
  | `graphics_iteration_step` | integer | Cycle interval between output dumps |
  
  **Field output lists:**
  
  | Field | Options |
  |---|---|
  | `elem_field_outputs` | `den`, `mass`, `pres`, `sie`, `sspd`, `stress` |
  | `node_field_outputs` | `coords`, `force`, `grad_level_set`, `mass`, `temp`, `vel` |
  | `mat_pt_field_outputs` | `den`, `eroded`, `mass`, `pres`, `sie`, `sspd`, `stress`, `volfrac` |
  | `gauss_pt_field_outputs` | `level_set`, `vel_grad`, `volume` |
  
  ---
  
</details>
  
<details>
  <summary><code>solver_options</code> : Physics solver configuration</summary>
  
  ---
  
  A list of solver blocks specifying which physics solvers to use.
  
  Each `solver` block contains:
  
  | Field | Type | Description |
  |---|---|---|
  | `method` | enum | Solver method (see below) |
  | `id` | integer | Unique solver identifier |
  | `time_end` | double | End time specific to this solver |
  | `use_moving_heat_source` | boolean | Enable moving heat source (thermal solvers only) |
  
  **Solver methods:**
  
  | Value | Description |
  |---|---|
  | `dynx_FE` | 3D Hydro |
  | `dynx_FE_rz` | 2D Hydro |
  | `level_set` | Level set solver |
  | `thrmex_FE` | Thermal solver |
  
  ---
  
</details>
  
<details>
    
  <summary><code>boundary_conditions</code> : Surface boundary conditions</summary>
  
  ---
  
  A list of `boundary_condition` blocks applied to geometric surfaces.
  
  **Surface definition (`surface:`):**
  
  | Field | Type | Description |
  |---|---|---|
  | `type` | enum | `cylinder`, `global`, `sphere`, `x_plane`, `y_plane`, `z_plane` |
  | `plane_position` | double | Position of the plane |
  | `radius` | double | Radius for cylinder/sphere |
  | `tolerance` | double | Geometric tolerance for surface selection |
  | `origin` | `[x, y, z]` | Surface origin |
  
  **BC models:**
  
  | Field | Options |
  |---|---|
  | `velocity_model` | `constant`, `fixed`, `none`, `piston`, `reflected`, `time_varying`, `user_defined` |
  | `stress_model` | `constant`, `global_contact`, `none`, `preload_contact`, `time_varying`, `user_defined`, `fracture` |
  | `temperature_model` | `constant`, `convection`, `none`, `radiation` |
  | `heat_flux_model` | string |
  
  **Parameter fields:**
  
  | Field | Type | Description |
  |---|---|---|
  | `solver_id` | integer | Solver this BC applies to |
  | `velocity_bc_global_vars` | list | Velocity model parameters |
  | `stress_bc_global_vars` | list | Stress model parameters |
  | `contact_bc_global_vars` | 2 ints | `[max_pair_solve_iters, max_global_solve_iters]` |
  | `temperature_bc_global_vars` | list | Temperature model parameters |
  | `heat_flux_bc_global_vars` | list | Heat flux model parameters |
  
  ---
  
</details>
  
<details>
  
  <summary><code>materials</code> : Material property definitions</summary>

  ---
  
  A list of `material` blocks defining material models.
  
  | Field | Type | Description |
  |---|---|---|
  | `id` | integer | Unique material identifier |
  | `tabular_model` | string | File path for tabular material models |
  
  **Equation of State (`eos_model`):**
  
  | Option | Description |
  |---|---|
  | `gamma_law_gas` | Gamma law gas EOS |
  | `linear_elastic_eos` | Linear elastic EOS |
  | `mie_gruneisen_eos` | Mie-Grüneisen EOS |
  | `host_user_defined_eos` | Host-side user-defined EOS |
  | `user_defined_eos` | User-defined EOS |
  | `no_eos` | No EOS |
  | `void` | Void material |
  
  `eos_model_type`: `coupled`, `decoupled`, `no_eos`
  
  **Strength (`strength_model`):**
  
  | Option |
  |---|
  | `hypo_elastic_plastic_strength` |
  | `hypo_elastic_plastic_strength_rz` |
  | `hypo_plasticity_strength` |
  | `hypo_plasticity_strength_rz` |
  | `host_ann_strength` |
  | `user_defined_strength` |
  | `no_strength` |
  
  `strength_model_type`: `increment_based`, `state_based`, `no_strength`
  
  **Dissipation (`dissipation_model`):**
  
  | Option |
  |---|
  | `MARS` |
  | `MARS_rz` |
  | `directional_MARS` |
  | `directional_MARS_rz` |
  | `no_dissipation` |
  
  **Erosion:**
  
  | Field | Type | Description |
  |---|---|---|
  | `erosion_model` | enum | `basic`, `no_erosion` |
  | `erode_tension_val` | double | Tension threshold for erosion |
  | `erode_density_val` | double | Density threshold for erosion |
  
  **Level Set:**
  
  | Field | Type | Description |
  |---|---|---|
  | `level_set_type` | string | Type of level set for this material |
  | `normal_velocity` | double | Normal velocity component |
  | `curvature_velocity` | double | Curvature-dependent velocity component |
  
  ---
  
</details>
  
<details>
  
  <summary><code>multimaterial_options</code> : Mixed-material element handling</summary>

  ---
  
  Configures options for elements containing multiple materials.
  
  | Field | Type | Description |
  |---|---|---|
  | `max_num_mats_per_element` | int/double | Max materials per element |
  
  **Material equilibration (`mat_equilibration_model`):**
  
  | Option | Description |
  |---|---|
  | `no_equilibration` | No equilibration |
  | `tipton` | Tipton equilibration |
  | `user_defined` | User-defined model |
  
  `mat_equilibration_global_vars`: list of parameters
  
  **Geometric equilibration (`geo_equilibration_model`):**
  
  | Option | Description |
  |---|---|
  | `no_equilibration` | No interface reconstruction |
  | `tipton` | Tipton model |
  | `user_defined` | User-defined model |
  
  `geo_equilibration_global_vars`: list of parameters
  
  ---
  
</details>
  
<details>
  
  <summary><code>regions</code> : Initial conditions and geometric region assignment</summary>

  ---
  
  A list of `region` blocks assigning material and state to geometric volumes.
  
  | Field | Type | Description |
  |---|---|---|
  | `solver_id` | integer | Solver this region belongs to |
  | `material_id` | integer | Material assigned to this region |
  
  **Volume definition (`volume:`):**
  
  | Field | Type | Description |
  |---|---|---|
  | `type` | enum | `box`, `cylinder`, `global`, `sphere`, `voxel_file`, `vtu_file` |
  | `file_path` | string | Path to voxel/vtu file |
  | `x1`, `x2`, `y1`, `y2`, `z1`, `z2` | double | Bounds for box regions |
  | `radius1`, `radius2` | double | Radii for cylinder/sphere |
  | `origin` | `[x, y, z]` | Volume origin |
  | `part_id` | integer | Part ID for multi-part meshes |
  
  **Distribution types** (shared across all field blocks below):
  
  `radial`, `spherical`, `tg_vortex`, `uniform`, `x_linear`, `y_linear`, `z_linear`
  
  **`volume_fraction:`**
  
  | Field | Description |
  |---|---|
  | `type` | Distribution type |
  | `value` | Constant or base value |
  | `slope` | Slope for linear distributions |
  | `origin` | Origin for radial/spherical |
  
  **`velocity:`**
  
  | Field | Description |
  |---|---|
  | `type` | `cartesian`, `radial`, `radial_linear`, `spherical`, `spherical_linear`, `static`, `tg_vortex` |
  | `u`, `v`, `w` | Cartesian velocity components |
  | `speed` | Speed magnitude for radial/spherical |
  | `value` | Uniform value |
  
  **Scalar initial condition fields** — all share `type`, `value`, (`slope`, `origin` where applicable):
  
  | Field |
  |---|
  | `temperature` |
  | `density` |
  | `specific_heat` |
  | `thermal_conductivity` |
  | `specific_internal_energy` |
  | `internal_energy` |
  | `level_set` |
  
</details>

---

## Example Input File

The following input YAML file showcases a 3D high-velocity impact and erosion simulation.

```yaml
dynamic_options:
  time_final: 10.0
  dt_min: 1.e-8
  dt_max: 1.e-3
  dt_start: 1.e-5
  cycle_stop: 300000

mesh_options:
  source: generate
  num_dims: 3
  type: box
  origin: [0.0, 0.0, 0.0]
  length: [1.2, 1.2, 1.2]
  num_elems: [30, 30, 30]

output_options:
  timer_output_level: thorough
  output_file_format: ensight
  graphics_time_step: 0.05

solver_options:
  - solver:
      method: dynx_FE
      id: 0

boundary_conditions:
  # Plane y=0: reflective
  - boundary_condition:
      solver_id: 0
      surface: 
        type: y_plane
        plane_position: 0.0
      velocity_model: reflected
      velocity_bc_global_vars: [0, 1, 0]

  # Plane z=0: reflective
  - boundary_condition:
      solver_id: 0
      surface: 
        type: z_plane
        plane_position: 0.0
      velocity_model: reflected
      velocity_bc_global_vars: [0, 0, 1]

materials:
  # ID 0: Copper-like Projectile
  - material:
      id: 0
      eos_model_type: decoupled
      eos_model: linear_elastic_eos
      eos_global_vars: [1.30, 0.44, 8.93, 0.001]
      strength_model_type: increment_based
      strength_model: hypo_plasticity_strength
      strength_global_vars: [0.44, 0.0021]
      dissipation_model: MARS
      dissipation_global_vars: [1.0, 1.333, 1.0, 1.333, 0.05, 1.0]

  # ID 1: Plate with Erosion
  - material:
      id: 1
      eos_model_type: decoupled
      eos_model: gamma_law_gas
      eos_global_vars: [1.666666666666667, 1.0E-4, 1.0]
      dissipation_model: MARS
      dissipation_global_vars: [1.0, 1.0, 1.333, 1.333, 0.1, 1.0]
      erosion_model: basic
      erode_tension_val: 2.0e-7
      erode_density_val: 5.0

  # ID 2: Background Air
  - material:
      id: 2
      eos_model_type: decoupled
      eos_model: gamma_law_gas
      eos_global_vars: [1.666666666666667, 1.0E-4, 1.0]
      dissipation_model: MARS
      dissipation_global_vars: [1.0, 1.0, 1.333, 1.333, 0.1, 1.0]

regions:
  # Global Air Fill
  - region:
      volume: {type: global}
      material_id: 2
      solver_id: 0
      density: {type: uniform, value: 0.010}
      specific_internal_energy: {type: uniform, value: 1.0e-6}
      velocity: {type: cartesian, u: 0.0, v: 0.0, w: 0.0}

  # Projectile
  - region:
      volume:
        type: box
        origin: [0.0, 0.0, 0.0]
        x1: 0.1, x2: 0.5, y1: 0.0, y2: 0.21, z1: 0.0, z2: 0.21
      material_id: 0
      solver_id: 0
      density: {type: uniform, value: 8.93}
      specific_internal_energy: {type: uniform, value: 1.0e-6}
      velocity: {type: cartesian, u: 0.3, v: 0.0, w: 0.0}

  # Wall/Plate
  - region:
      volume:
        type: box
        origin: [0.0, 0.0, 0.0]
        x1: 0.5, x2: 0.6, y1: 0.0, y2: 1.2, z1: 0.0, z2: 1.2
      material_id: 1
      solver_id: 0
      density: {type: uniform, value: 8.93}
      specific_internal_energy: {type: uniform, value: 1.0e-6}
      velocity: {type: cartesian, u: 0.0, v: 0.0, w: 0.0}
```

---

## Running Fierro

### Fierro Environment Setup

Fierro and its dependencies (ROCM, HIP, Kokkos) are all managed by the Fierro module. Load the module before running or submitting any job.

```bash
ml fierro
```

After loading the module, this makes the `Fierro` binary available in your `PATH` and setups the required HIP/ROCM environment. You can verify it's loaded correctly with:

```bash
which Fierro
```

---

### Submitting a Job

The recommended way to run Fierro is using a SBATCH script. The following SBATCH script is a simple template for Fierro which should be able to run any kind of Fierro simulation:

```bash
#!/bin/bash
#SBATCH --job-name="fierro-demo`
#SBATCH --nodes=1
#SBATCH --ntask-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --gres=gpu
#SBATCH --chdir=<path/to/dir>
#SBATCH --time=0-0:30:00
#SBATCH --output=-%j.out

module load fierro

srun Fierro input.yaml
```

Replace `input.yaml` with either the example input file from ealier or you're own YAML Fierro input file. 
Then, you're ready to run the script with `sbatch <scriptName>`.

```{tip}
**Output Directory:** Fierro creates a subdirectory in the working directory names after the `output_file_format` value in your input file. For example, `output_file_format: vtk` produces a `vtk/` folder. Setting `--chdir` ensures output goes in a certain directory rather than where you called the SBATCH file.
```

---

## Viewing the Output with Paraview

### Paraview Environment Setup

To view the output of your Fierro script, first you need to load the Paraview module.

```bash
ml contrib paraview
```

To run paraview just simply do `paraview` in the terminal. It will pop up a gui window of paraview that you can use to view the output file on Muscadine

---

### Opening Output Files

Fierro outputs the file in a directory named after whatever your `output_file_format` was (e.g. `vtk/`).

1. In ParaView: **File -> Open**
2. Navigate to the directory you set for `--chdir` in your SBATCH script and open the new output directory.
3. Look for a `.pvd` file
4. Select it and click **OK**
5. Click the green **Apply** button in the Properties panel

---

### Basic Workflow

**Scrub through timesteps:**
Use the toolbar at the top - playbutton, step/forward/back, or type a timestep directly in the time field.

**Color by a field:** 
In the toolbar, change the coloring dropdown from `Solid Color` to a field from your `output_options` - e.g. `den`, `pres`, `vel`. Hit **Apply** after changing.

**Rescale the colormap to the current timesetp:**
Click the **Rescale to Data Range** button (the double-arrow icon next to the colormap bar) - useful when fields change dramatically over time.

**Warp by velocity (optional):**
**Filters -> Search -> "Warp By Vector"** -> set vector to `vel` -> Apply. Useful for visualizing deformation.

```{tip}
For large runs, load only a subset of timesteps: **File → Open** → check **"Load only selected timesteps"** in the dialog.
```

---













