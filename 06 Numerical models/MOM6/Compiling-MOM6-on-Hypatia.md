# Compiling MOM6 on Hypatia
*Contributors: Graeme MacGilchrist, CODEX*

This is a wiki-style `mkmf` recipe for compiling MOM6 and MOM6-SIS coupled model configurations. It incorporates a working module stack, complete FMS discovery, consistent NetCDF/HDF5 linkage, and safe smoke-test launch for the coupled model.

The compilation relies on a module `netcdf-fortran/test-4.6.2` to get around some inconsistencies with other installs of NetCDF and HDF libraries. It is not clear whether this "test" module environment will be retained. Thus if this compile recipe fails in the future, that may be the reason.

Before proceeding with compiling, copy the following `mkmf` template into the relevant folder in the `MOM6-examples` repository: `/home/gam24/pkgs/MOM6-examples/src/mkmf/templates/hypatia-gnu.mk`.

## 1. Load the environment

Run everything in the same shell:

```
cd /home/gam24/pkgs/MOM6-examples

module purge
module load gnu/14.2.0
module load openmpi/5.0.7
module load netcdf-fortran/test-4.6.2
```

Note: Ensure that the following modules are **NOT** additionally loaded as they cause conflicts:

```
netcdf/4.9.3
netcdf-fortran/4.6.2
hdf5/1.14.6
```

The `test-4.6.2` module loaded above supplies a coherent NetCDF-C, NetCDF-Fortran, HDF5, and MPI-compatible stack.

Set the build environment:

```
export CC=mpicc
export FC=mpifort
export MPICC=mpicc
export MPIFC=mpifort
export OMP_NUM_THREADS=1

export MOM6_ROOT="$(pwd)"
export MOM6_LINK_FLAGS="$(nf-config --flibs)"
export BUILD_JOBS=8
```

Verify it:

```
module -t list

command -v mpicc
command -v mpifort
command -v nc-config
command -v nf-config
command -v h5pcc

nf-config --fflags
nf-config --flibs
```

The NetCDF tools should resolve beneath:

```
/software/apps/netcdf-stack/4.9.3/
```

## 2. (Optional) Clean Build

For a completely clean rebuild:

```
rm -rf build/fms
rm -rf build/ocean_only
rm -rf build/ice_ocean_SIS2
```

These commands remove generated build products only.

## 3. Compile FMS

Generate its source manifest:

```
mkdir -p build/fms
cd build/fms

../../src/mkmf/bin/list_paths -l ../../src/FMS
```

Generate the Makefile:

```
../../src/mkmf/bin/mkmf \
  -t ../../src/mkmf/templates/hypatia-gnu.mk \
  -p libfms.a \
  -c "-Duse_libMPI \
      -Duse_netCDF \
      -DSPMD \
      -DUSE_LOG_DIAG_FIELD_INFO \
      -DMAXFIELDMETHODS_=500 \
      -Duse_AM3_physics" \
  path_names
```

Compile:

```
make -j"${BUILD_JOBS}" \
  REPRO=1 \
  FC=mpifort \
  CC=mpicc \
  LD=mpifort \
  libfms.a
```

Verify:

```
ls -lh libfms.a
```

Return to the repository root:

```
cd "${MOM6_ROOT}"
```

## 4. Compile Ocean-Only MOM6

Generate its source manifest:

```
mkdir -p build/ocean_only
cd build/ocean_only

../../src/mkmf/bin/list_paths -l \
  ../../src/MOM6/config_src/infra/FMS1 \
  ../../src/MOM6/config_src/memory/dynamic_symmetric \
  ../../src/MOM6/config_src/drivers/solo_driver \
  ../../src/MOM6/config_src/external \
  ../../src/MOM6/src \
  ../../src/FMS/coupler/atmos_ocean_fluxes.F90
```

Generate the Makefile:

```
../../src/mkmf/bin/mkmf \
  -t ../../src/mkmf/templates/hypatia-gnu.mk \
  -o "-I../fms" \
  -p MOM6 \
  -l "-L../fms -lfms ${MOM6_LINK_FLAGS}" \
  -c "-Duse_libMPI \
      -Duse_netCDF \
      -DSPMD \
      -DUSE_LOG_DIAG_FIELD_INFO \
      -DMAXFIELDMETHODS_=500 \
      -Duse_AM3_physics" \
  path_names
```

Compile:

```
make -j"${BUILD_JOBS}" \
  REPRO=1 \
  FC=mpifort \
  CC=mpicc \
  LD=mpifort \
  MOM6
```

Verify:

```
ls -lh MOM6
```

Return to the repository root:

```
cd "${MOM6_ROOT}"
```

## 5. Compile Coupled MOM6–SIS2

This compiles MOM6, SIS2, the coupler, null atmosphere and land models, ice parameters, and icebergs directly into one executable.

```
mkdir -p build/ice_ocean_SIS2
cd build/ice_ocean_SIS2

../../src/mkmf/bin/list_paths -l \
  ../../src/MOM6/config_src/infra/FMS1 \
  ../../src/MOM6/config_src/memory/dynamic_symmetric \
  ../../src/MOM6/config_src/drivers/FMS_cap \
  ../../src/MOM6/config_src/external \
  ../../src/SIS2/config_src/dynamic_symmetric \
  ../../src/SIS2/config_src/external \
  ../../src/MOM6/src \
  ../../src/SIS2/src \
  ../../src/atmos_null \
  ../../src/coupler \
  ../../src/land_null \
  ../../src/ice_param \
  ../../src/icebergs/src \
  ../../src/FMS/coupler \
  ../../src/FMS/include
```

Generate the Makefile:

```
../../src/mkmf/bin/mkmf \
  -t ../../src/mkmf/templates/hypatia-gnu.mk \
  -o "-I../fms" \
  -p MOM6 \
  -l "-L../fms -lfms ${MOM6_LINK_FLAGS}" \
  -c "-Duse_libMPI \
      -Duse_netCDF \
      -DSPMD \
      -DUSE_LOG_DIAG_FIELD_INFO \
      -DMAXFIELDMETHODS_=500 \
      -Duse_AM3_physics \
      -D_USE_LEGACY_LAND_" \
  path_names
```

Compile:

```
make -j"${BUILD_JOBS}" \
  REPRO=1 \
  FC=mpifort \
  CC=mpicc \
  LD=mpifort \
  MOM6
```

Verify:

```
ls -lh MOM6
```

Return to the repository root:

```
cd "${MOM6_ROOT}"
```

## 6. Coupled Smoke Test

Create a fresh run directory for the two-rank Baltic example:

```
export RUN_DIR="${MOM6_ROOT}/run/ice_ocean_SIS2/Baltic/smoke"
export MOM6_EXE="${MOM6_ROOT}/build/ice_ocean_SIS2/MOM6"

mkdir -p "${RUN_DIR}"
cp -a "${MOM6_ROOT}/ice_ocean_SIS2/Baltic/." "${RUN_DIR}/"
```

Link datasets to new run folder:

```
cd "${RUN_DIR}"
ln -sf "${MOM6_ROOT}"/.datasets/ INPUT/.datasets
```

Run locally, if login-node MPI execution is permitted:

```
cd "${RUN_DIR}"
mpirun -np 2 "${MOM6_EXE}"
```

Or use Slurm:

```
salloc --nodes=1 --ntasks=2 --time=00:30:00

cd "${RUN_DIR}"
srun -n 2 "${MOM6_EXE}"
```

Check the result:

```
tail -n 20 ocean.stats
ls -lh *.nc
```

Use the same module environment for compilation and every subsequent model run.