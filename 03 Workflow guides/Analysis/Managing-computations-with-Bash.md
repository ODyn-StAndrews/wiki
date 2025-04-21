# Managing computations with `bash`

We'll go through a working example, and detail the meaning of every relevant subpart of the code.

For this example, we consider that we have a Python script, called `generate_input_for_MOM6.py`, that generates a bunch of [NetCDF](https://www.unidata.ucar.edu/software/netcdf/) files to be given as input to (e.g.) MOM6.

This Python script could look like this:

```python
### Import all the required modules
import numpy as np
import xarray as xr

### Generate grid

lat_north=-30.0 ; lon_west=0.0   # Domain boundaries (in degrees north and east)
lat_south=-70.0 ; lon_east=80.0
max_depth=4000.0                 # Maximum depth (in meters)

grid = xr.Dataset()
# Do some complex stuff
grid.to_netcdf('grid.nc')

## Generate synthetic surface forcings

max_wind_stress=0.15 # Maximum surface wind stress (in Pa)
max_heat_flux=20     # Maximum heat flux (in W/m2)

wind_stress = xr.Dataset()
heat_flux = xr.Dataset()
# Do some complex stuff
wind_stress.to_netcdf('wind_stress.nc')
heat_flux.to_netcdf('heat_flux.nc')
```
This Python script generates 3 NetCDF files, one containing the full grid data, and the other two containing the surface forcing data.
While the grid specification might not change much from one computation to another, forcing parameters are something that we want to be able to change easily in order to study the response of the model to these changing parameters, and this is where Bash scripts come into play.

The first step is to create a template Python file, say `generate_input_for_MOM6.py.temp` that is exactly the same as the one above, but where numerical values of parameters are replaced by keywords, and which would look like this:

```python
### Import all the required modules
import numpy as np
import xarray as xr

### Generate grid

lat_north=LAT_NORTH ; lon_west=LON_WEST   # Domain boundaries (in degrees north and east)
lat_south=LAT_SOUTH ; lon_east=LON_EAST
max_depth=MAX_DEPTH                 # Maximum depth (in meters)

grid = xr.Dataset()
# Do some complex stuff
grid.to_netcdf('grid.nc')

## Generate synthetic surface forcings

max_wind_stress=MAX_WIND_STRESS # Maximum surface wind stress (in Pa)
max_heat_flux=MAX_HEAT_FLUX     # Maximum heat flux (in W/m2)

wind_stress = xr.Dataset()
heat_flux = xr.Dataset()
# Do some complex stuff
wind_stress.to_netcdf('wind_stress.nc')
heat_flux.to_netcdf('heat_flux.nc')
```
See how e.g. `max_wind_stress=0.15` was replaced by `max_wind_stress=MAX_WIND_STRESS`.

The Bash script will submit jobs by running `sbatch run_MOM6.slurm`, and this file (which incidentally is a Bash script) could look like this:

```bash
#!/bin/bash

#SBATCH --job-name=SO_channel
#SBATCH --time=8:00:00
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --account=e786
#SBATCH --partition=standard
#SBATCH --qos=standard

export OMP_NUM_THREADS=1 #   This prevents any threaded system libraries from automatically using threading.
export SRUN_CPUS_PER_TASK=$SLURM_CPUS_PER_TASK # Propagate the cpus-per-task setting from script to srun commands

# Launch the parallel job
srun --distribution=block:block --hint=nomultithread /work/e786/e786/npoumaere/MOM6-examples/build/ocean_only/MOM6
```

Now for the Bash script proper. It first sets some static parameters, then goes through a loop for changing parameters. For each set of parameters, it first creates a running directory, then creates the required Python script, runs it with `python3` and finally submits the job to SLURM, which job calls the required MOM6 executable.

Let's first have a look at the whole script, and we'll come back to it in details afterwards:
```bash
#!/bin/bash

#########################################
#   DOMAIN DEFINITION (does not change)
#########################################

LAT_NORTH=-30.0
LAT_SOUTH=-70.0
LON_WEST=0.0
LON_EAST=80.0
MAX_DEPTH=4000.0

#########################################
#   FORCING PARAMETERS (do change)
#########################################

for MAX_WIND_STRESS in 0.05 0.10 0.15
do
for MAX_HEAT_FLUX in 10.0 20.0
do

#TODO: tuples of parameters

RUNDIR="run_tau=${MAX_WIND_STRESS}_heat=${MAX_HEAT_FLUX}"

mkdir -p $RUNDIR
cd $RUNDIR

TEMPFILE="../generate_input_for_MOM6.py.temp"
IFILE="generate_input_for_MOM6.py"

cat $TEMPFILE \
| sed s/LAT_NORTH/$LAT_NORTH/g \
| sed s/LAT_SOUTH/$LAT_SOUTH/g \
| sed s/LON_WEST/$LON_WEST/g \
| sed s/LON_EAST/$LON_EAST/g \
| sed s/MAX_DEPTH/$MAX_DEPTH/g \
| sed s/MAX_WIND_STRESS/$MAX_WIND_STRESS/g \
| sed s/MAX_HEAT_FLUX/$MAX_HEAT_FLUX/g  \
> $IFILE

python3 $IFILE

# TODO: show what type of calculation is doable by bash

sbatch run_MOM6.slurm

cd ..

done
done

```

The very first line `#!/bin/bash` simply stipulates which shell will interpret the file (could also be `sh`, `csh`, `ksh`).

As said above, geometric parameters won't change in our parametric study, so we define them first and before any loop.

```bash
LAT_NORTH=-30.0
LAT_SOUTH=-70.0
LON_WEST=0.0
LON_EAST=80.0
MAX_DEPTH=4000.0
```

Then we want to change the maximum wind stress through three values, and the maximum heat flux through two values. This is done through a "for" loop, which syntax is:
```bash
for parameters in value1 value2 value3
do 

# Do some stuff
# TODO: show C syntax for loops

done
```
The two loops are nested so that every possible combination of parameters will be run (but parameters tuples can be used to avoid that).

At each iteration of the (nested) loop, we create the run folder that corresponds to these parameters thanks to the `mkdir` command (with `-p` option which avoids raising an error if directory already exists). The name of the folder is automatically related to the parameters thanks to the `${}` syntax. We then go to this folder with the `cd` command, and specify the name of the template (existent) and actual (about to be created) Python scripts. Note that these names are specified in the loop for the sake of generality, but this could have be done before the loop as they do not change.

This all corresponds to this bit of code:

```bash
RUNDIR="run_tau=${MAX_WIND_STRESS}_heat=${MAX_HEAT_FLUX}"

mkdir -p $RUNDIR
cd $RUNDIR

TEMPFILE="../generate_input_for_MOM6.py.temp"
IFILE="generate_input_for_MOM6.py"
```

Then, we can take our template Python file (which is in the parent folder, hence the `../` above), and use the `sed` command to replace the parameter keywords with their required value.
The command that "takes" the template script is `cat`, which in its most simple use outputs the entirety of a (text) file to the terminal.
What we do here is redirect this output as input to the `sed` command. This is done through the pipe `|` command.
Here the syntax `sed s/PARAM_KEYWORD/$PARAM/g` translates as: search all instances of the word `PARAM_KEYWORD` in the file and replace them by the value of `$PARAM` specified by the loop. Multiple pipes allow us to do that for multiple parameters. 
The last line `> $IFILE` writes the output of this whole bit to the actual Python file: 
```bash
cat $TEMPFILE \
| sed s/LAT_NORTH/$LAT_NORTH/g \
| sed s/LAT_SOUTH/$LAT_SOUTH/g \
| sed s/LON_WEST/$LON_WEST/g \
| sed s/LON_EAST/$LON_EAST/g \
| sed s/MAX_DEPTH/$MAX_DEPTH/g \
| sed s/MAX_WIND_STRESS/$MAX_WIND_STRESS/g \
| sed s/MAX_HEAT_FLUX/$MAX_HEAT_FLUX/g  \
> $IFILE
```
The `>` means that the file is created if it does not exist and is written over if it does (be cautious!)

We then run Python on this file (`python3 $IFILE`), submit the job to SLURM (`sbatch run_MOM6.slurm`), and go back to the parent folder with the `cd ..` command.  

