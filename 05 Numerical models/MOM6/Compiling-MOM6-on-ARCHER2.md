# Compiling
- General info on [[MOM6]]

# Compiling on ARCHER2

Following the general instructions on the [MOM6 wiki](https://github.com/NOAA-GFDL/MOM6-examples/wiki/Getting-started).

- Info on [[ARCHER2]]

## Dataset downloads

Download datasets to ARCHER2 with 
```
wget -r ftp://ftp.gfdl.noaa.gov/perm/Alistair.Adcroft/MOM6-testing/
```
unpack them, then link them with
```
cd MOM6-examples # navigate to base directory
ln -sf /path/to/downloaded/data .datasets
```
## `mkmf` approach
Necessary for anything other than `ocean-only` configurations (`ocean-only` configurations can be built with the `auto-configure` approach, though I've not tried this on ARCHER2)

The compilation environment should be established with the following module calls:
```
module load PrgEnv-gnu  
module load cray-hdf5  
module load cray-netcdf  
module load cmake
```
Per the MOM6 wiki [here](https://github.com/NOAA-GFDL/MOM6-examples/wiki/Getting-started#setting-up-the-compile-environment), this can be included in a `env` file if desired, using
```
mkdir build
cat > build/env << EOFA
module load PrgEnv-gnu  
module load cray-hdf5  
module load cray-netcdf  
module load cmake
EOFA
```
and load these modules with `source build/env`.
### `mkmf` template modifications
Compiling with `mkmf` requires the use of certain templates, found in `src/mkmf/templates`, for which we will use `linux-gnu.md`. A file containing the necessary modifications to this template can be found at `/work/e786/shared/graemem/MOM6-examples/src/mkmf/templates/archer-gnu.mk`.

The specific modifications are:
1. Set the compiler and library names to the default wrappers on ARCHER2 (see [here](https://docs.archer2.ac.uk/quick-start/quickstart-developers/)). Do this on lines 8-11, which should now read:
```
############
# Commands Macros
FC = ftn
CC = cc
CXX = CC
LD = ftn $(MAIN_PROGRAM)
```
2. Add the following flags to the Fortran compiler flags (line 95):
```
-fallow-invalid-boz -fallow-argument-mismatch
```
### Compile `FMS`
In creating the `Makefile` one additional flag is required: `-DHAVE_GETTID`. The set of commands now reads (pointing to the modified template file):
```
mkdir -p build/fms/
(cd build/fms; rm -f path_names; \
../../src/mkmf/bin/list_paths -l ../../src/FMS; \
../../src/mkmf/bin/mkmf -t ../../src/mkmf/templates/archer-gnu.mk -p libfms.a -c "-Duse_libMPI -Duse_netCDF -DHAVE_GETTID" path_names)
```
This generates the `Makefile`. Compile with:
```
(cd build/fms/; source ../env; make NETCDF=3 REPRO=1 libfms.a -j)
```
### Compile `MOM6`
Generate the `Makefile` :
```
mkdir -p build/ocean_only/
(cd build/ocean_only/; rm -f path_names; \
../../src/mkmf/bin/list_paths -l ./ ../../src/MOM6/{config_src/infra/FMS1,config_src/memory/dynamic_symmetric,config_src/drivers/solo_driver,config_src/external,src/{*,*/*}}/ ; \
../../src/mkmf/bin/mkmf -t ../../src/mkmf/templates/archer-gnu.mk -o '-I../fms' -p MOM6 -l '-L../fms -lfms' path_names)
```
Compile with:
```
(cd build/ocean_only/; source ../env; make REPRO=1 MOM6 -j)
```
### Compile `MOM6-SIS2`
Generate the `Makefile`:
```
mkdir -p build/ice_ocean_SIS2/
(cd build/ice_ocean_SIS2/; rm -f path_names; \
../../src/mkmf/bin/list_paths -l ./ ../../src/MOM6/config_src/{infra/FMS1,memory/dynamic_symmetric,drivers/FMS_cap,external} ../../src/SIS2/config_src/dynamic_symmetric ../../src/MOM6/src/{*,*/*}/ ../../src/{atmos_null,coupler,land_null,ice_param,icebergs/src,SIS2,FMS/coupler,FMS/include}/)
(cd build/ice_ocean_SIS2/; \
../../src/mkmf/bin/mkmf -t ../../src/mkmf/templates/archer-gnu.mk -o '-I../fms' -p MOM6 -l '-L../fms -lfms' -c '-Duse_AM3_physics -D_USE_LEGACY_LAND_' path_names )
```
Compile with:
```
(cd build/ice_ocean_SIS2/; source ../env; make REPRO=1 MOM6 -j)
```

## Run a test
Just to check that the compilation has worked, run one of the test configurations via an interactive job. First, start the job:
```
salloc --nodes=8 --ntasks-per-node=128 --cpus-per-task=1 --time=00:20:00 --partition=standard --qos=short
```
Then, run the test:
```
cd ocean_only/double_gyre/
mkdir -p RESTART
srun -n 8 ../../build/ocean_only/MOM6
```

# Compiling on a Mac
This information is derived from compiling the model on a 2018 MacBook Pro, running OSX 13.2 (Ventura).

## `autoconf` approach
Detailed instructions [here](https://github.com/NOAA-GFDL/MOM6/tree/dev/gfdl/ac).

*Preliminary steps*:
1. Deactive any `conda` environment. Details of setting up an appropriate `conda` environment can be found [here](https://github.com/NOAA-GFDL/MOM6-examples/wiki/Compiling-on-MacBook-M1-chip-using-conda-environment). For this set-up, I am installing everything in the native user environment (i.e. not `conda`). **Note that you need to deactivate `conda` each time before you create the Makefile using `autoconf`, otherwise it will pick up some of the associated `conda` packages and lead to errors**.
2. Set up environment, using `homebrew` to install required dependencies (see dependencies [here](https://github.com/NOAA-GFDL/MOM6-examples/wiki/MacOS-Compiler-Environment#dependencies)):
```
brew install make
brew install gfortran
brew install mpich
brew install netcdf
brew install netcdf-fortran
brew install autoconf
brew install automake
```
(Ensure that command line tools (`xcode`) are up-to-date. I ran `sudo xcode-select --install` to ensure this was the case.)

*Compile steps*
1. Compile FMS
    1. Issue `make -j` in `src/MOM6/ac/deps`. This clones and compiles `FMS` and builds it in the local directory.
2. Create configuration script
    1. Issue `autoreconf` in `src/MOM6/ac`. Creates `configure` script in local directory.
3. Compile the model
    1. Create `build` directory (preferable to do this in `MOM6-examples/build`).
    2. Execute `configure` script (found in `src/MOM6/src/ac`) to create a `Makefile`.
    3. Issue `make -j` in `build` directory to compile.