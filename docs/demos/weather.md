# Weather Simulations



## NeuralGCM

```{video} ../_static/figures/demos/temp-1.mp4
:align: center
:nocontrols:
:autoplay:
:loop:
:width: 100%
:playsinline:
```

NeuralGCM is a project to deploy hybid ML/Physics models for weather and climate simulations. In this demo guide, we will explore running their pre-trained temperature/humitidy, precipitation, and evaporation models. To learn more about NeuralGCM, visit their documentation and take a gander at their paper below.

- NeuralGCM Documentation: [LINK](https://neuralgcm.readthedocs.io/en/latest/index.html)
- NeuralGCM Paper: [LINK](https://arxiv.org/pdf/2412.11973)


### Installing NeuralGCM and Dependencies

This guide follows the install process of AMD's ROCm AI Ecosystem environment. This can be found at the following link: [LINK](https://rocm.docs.amd.com/projects/ai-ecosystem/en/latest/index.html). Specifically, NeuralGCM uses JAX.

First `cd` into the directory of your choosing. We recommend doing this in either `/work` or `/scratch`. Module load pyhton3.14 and make a python virtual environment in your desired environment.

```bash
ml load python/3.14
python -m venv .venv
source .venv/bin/activate
```

Now we are going to install the ROCm python libraries for our specific GPU. Run the following to do so:

```bash
python -m pip install --index-url https://repo.amd.com/rocm/whl-multi-arch/ \
    "rocm[libraries,device-gfx90a]"
```

Now that the ROCm libraries have been installed, we will install JAX and other necessary libraries. Run the following to install JAX:

```bash
python -m pip install --index-url https://repo.amd.com/rocm/whl-multi-arch/ \
    "jax_rocm7_plugin==0.10.0+rocm7.14.0" \
    "jax_rocm7_pjrt==0.10.0+rocm7.14.0"

# Install jax from PyPI
python -m pip install \
    "jax==0.10.0" \
    "jaxlib==0.10.0"

# Install additional requirements
python -m pip install \
    xarray \
    matplotlib \
    dinosaur \
    cartopy \ 
    neuralgcm
```

To ensure JAX was properly install, run the following test script, you should see `[RocmDevice(id=0)]`.

```bash
python -c "import jax; print(jax.devices())"
```


### Choosing a Model

Since there are so few pre-trained models and they are small in size, we provide all of NeuralGCM's pre-trained models in `/work/models/NeuralGCM`. You'll notice that most of the models are 2.8 degree and 1.4 degree with one being 0.7 degree. These numbers represent the resolution of the outputs. The larger the degree size, the courser the output will appear, but there are side effects of choosing a higher resolution model. Both the 1.4 and 0.7 models will not produce accurate long-term results. The general rule of thumb for simulation time is included in the table below along.

| Model                                        | Results       | Resolution    | Max Simulation Time | Min Timestep |
| -------------------------------------------- | ------------- | ------------- | ------------------- | ------------ |
| v1/deterministic\_0\_7\_deg.pkl              | Temp/Humidity | 78km x 78km   | 7 days              | 1 hour       |
| v1/deterministic\_1\_4\_deg.pkl              | Temp/Humidity | 156km x 156km | 6 months            | 1 hour       |
| v1/deterministic\_2\_8\_deg.pkl              | Temp/Humidity | 312km x 312km | 1 year              | 1 hour       |
| v1/stochastic\_1\_4\_deg.pkl                 | Temp/Humidity | 156km x 156km | 6 months            | 1 hour       |
| v1\_precip/stochastic\_precip\_2\_8\_deg.pkl | Precipitation | 312km x 312km | 1 year              | 6 hours      |
| v1\_precip/stochastic\_evap\_2\_8\_deg.pkl   | Evaporation   | 312km x 312km | 1 year              | 6 hours      |

It should be known that these models *supposedly* are said to be stable for longer than what we are reporting, and that is because of our own testing on our hardware. They used and trained on Google TPUs which are specifically designed for these sort of tasks.

These are just the pre-trained models though, there is support for you to train your own model or even fine-tune their models on the data you want to use. Reference their documentation and paper to learn more about how to train a NeuralGCM model.

### Choosing a Dataset

Due to Muscadine's lack of *direct* internet access, large datasets such as ERA5 data must be downloaded by one of the student sysadmins. Please reach out to your current student sysadmin for updated/specific ERA5 data.

Muscadine will have several locally stored ERA5 datasets to use for weather demos and tests. These datasets can be found at `/scratch/shared/ERA5/`. These datasets will be named according to the following naming scheme: `YYYY-MM-DD\_YYYY-MM-DD.zarr`.

### Basic Inference Script

Since NeuralGCM is a python library and takes advantage of a Python ML library called JAX, our script will be in Python. Below you'll find the bare minimum to load the model and dataset, inference, and process a small mp4 of the results.
```python
import jax
import numpy as np
import pickle
import xarray
from dinosaur import horizontal_interpolation
from dinosaur import spherical_harmonic
from dinosaur import xarray_utils
import neuralgcm

# Parameters | CHANGE THESE
model_name = "v1/deterministic_2_8_deg.pkl"
era5_path = "/scratch/shared/ERA5/2020-02-14_2020-02-18.zarr"
demo_start_time = '2020-02-14'
demo_end_time = '2020-02-18'
inner_steps = 24 # in hours


with open(f"/work/models/NeuralGCM/{model_name}", "rb") as f:
        ckpt = pickle.load(f)
model = neuralgcm.PreasureLevelModel.from_checkpoint(ckpt)

data_inner_steps = 24
full_era5 = xarray.open_zarr(era5_path, chunks={"time": 24})
sliced_era5 = (
    full_era5
    [model.input_variables + model.forcing_variables]
    .pipe(
        xarray_utils.selective_temporal_shift,
        variables=model.forcing_variables,
        time_shift='24 hours',
    )
    .sel(time=slice(demo_start_time, demo_end_time, data_inner_steps))
    .compute()
)

era5_grid = spherical_harmonic.Grid(
    latitude_nodes=full_era5.sizes['latitude'],
    longitude_nodes=full_era5.sizes['longitude'],
    latitude_spacing=xarray_utils.infer_latitude_spacing(full_era5.latitude),
    longitude_offset=xarray_utils.infer_longitude_offset(full_era5.longitude),
)
regridder = horizontal_interpolation.ConservativeRegridder(
    era5_grid, model.data_coords.horizontal, skipna=True
)
eval_era5 = xarray_utils.regrid(sliced_era5, regridder)
eval_era5 = xarray_utils.fill_nan_with_nearest(eval_era5)

outer_steps = 4 * 24 // inner_steps  # total of 4 days
timedelta = np.timedelta64(1, 'h') * inner_steps
times = (np.arange(outer_steps) * inner_steps)  # time axis in hours

# Initialize model state
inputs = model.inputs_from_xarray(eval_era5.isel(time=0))
input_forcings = model.forcings_from_xarray(eval_era5.isel(time=0))
rng_key = jax.random.key(42)  # optional for deterministic models
initial_state = model.encode(inputs, input_forcings, rng_key)

# Use persistence for forcing variables (SST and sea ice cover)
all_forcings = model.forcings_from_xarray(eval_era5.head(time=1))

# Make forecast
final_state, predictions = model.unroll(
    initial_state,
    all_forcings,
    steps=outer_steps,
    timedelta=timedelta,
    start_with_input=True,
)
predictions_ds = model.data_to_xarray(predictions, times=times)

# Selecting ERA5 targets from exactly the same time slice
target_trajectory = model.inputs_from_xarray(
    eval_era5
    .thin(time=(inner_steps // data_inner_steps))
    .isel(time=slice(outer_steps))
)
target_data_ds = model.data_to_xarray(target_trajectory, times=times)

combined_ds = xarray.concat([target_data_ds, predictions_ds], 'model')
combined_ds.coords['model'] = ['ERA5', 'NeuralGCM']

# Visualize ERA5 vs NeuralGCM trajectories
fg = combined_ds.specific_humidity.sel(level=850).plot(
    x='longitude', y='latitude', row='time', col='model', robust=True, aspect=2, size=2
);

# Save to png
fg.fig.savefig('neuralgcm-demo.png', dpi=400)
```

### Advanced Provided Script

We understand that there are many different configurations, settings, and such to change anytime you want to rerun NeuralGCM. Because of this, we provide a simple to use python script along with accompanying sbatch script.

To access these scripts, run the following:
```bash
ml load neuralgcm
```

This will load the scripts you'll need to run a weather simulation. With the neuralgcm module loaded, run the following to submit a slurm job in your directory.

```{attention}
You would of needed to complete the steps found in "Installing NeuralGCM and Dependencies" and to be in said directory for the script to function properly. Also note that the script will store slurm logs in `$PWD/logs` and figures in `$PWD/plots`.
```

```bash
sbatch neuralgcm
```

| Parameter       | Description                           | Data Type      | Default    | 
| --------------- | ------------------------------------- | -------------- | ---------- |
| -h, --help      | show the help message and exit        | N/A            | N/A        |
| --mode          | What mode to run the script in        | string{sup}`*` | temp       |
| --duration      | How long to simulate (in days)        | int            | 7          |
| --step-size     | Simulation step size (in hours)       | int            | 3          |
| --climate-model | Which climate model to run            | string         | {sup}`**`  |
| --era5-path     | Path to the desired ERA5 dataset      | string         | {sup}`***` |
| --fps           | Frames Per Second to render animation | int            | 12         |
| --dpi           | Dots Per Inch to render animation     | int            | 200        |
| --format        | Option between "gif" or "mp4"         | string         | mp4        |

- {sup}`*` Options include: demo, full, precip, temp
- {sup}`**` Default climate model is `v1/deterministic_2_8_deg.pkl`
- {sup}`***` Default ERA5 dataset is `/scratch/shared/ERA5/2020-02-14_2020-02-18.zarr`

