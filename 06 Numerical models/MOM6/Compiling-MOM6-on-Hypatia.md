# Compiling MOM6 on Hypatia
*Contributors: Graeme MacGilchrist, CODEX*

This is a wiki-style `mkmf` recipe for compiling MOM6 and MOM6-SIS coupled model configurations. It incorporates a working module stack, complete FMS discovery, consistent NetCDF/HDF5 linkage, and safe smoke-test launch for the coupled model.

The compilation relies on a module `netcdf-fortran/test-4.6.2` to get around some inconsistencies with other installs of NetCDF and HDF libraries. It is not clear whether this "test" module environment will be retained. Thus if this compile recipe fails in the future, that may be the reason.

Before proceeding with compiling, you need to inlude a `hypatia`-specific `mkmf` template in your MOM6-examples repository (in the following location: `~/MOM6-examples/src/mkmf/templates/hypatia-gnu.mk`). You can do so by copying the version from here: `/home/gam24/pkgs/MOM6-examples/src/mkmf/templates/hypatia-gnu.mk`. Or, you can create a new file in the relevant folder and copy in the text found at the bottom of this note.

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

# `mkmf` template file
Copy the following text into a file called `hypatia-gnu.mk` within the MOM6-examples `mkmf` templates folder (`~/MOM6-examples/src/mkmf/templates/`).

```bash
# Template for the GNU Compiler Collection on a Cray System
#
# Typical use with mkmf
# mkmf -t ncrc-cray.mk -c"-Duse_libMPI -Duse_netCDF" path_names /usr/local/include

############
# Commands Macros
############
# Default to MPI compiler wrappers so builds automatically link MPI libs.
FC ?= mpifort
CC ?= mpicc
LD ?= $(FC) $(MAIN_PROGRAM)

#######################
# Build target macros
#
# Macros that modify compiler flags used in the build.  Target
# macrose are usually set on the call to make:
#
#    make REPRO=on NETCDF=3
#
# Most target macros are activated when their value is non-blank.
# Some have a single value that is checked.  Others will use the
# value of the macro in the compile command.

DEBUG =              # If non-blank, perform a debug build (Cannot be
                     # mixed with REPRO or TEST)

REPRO =              # If non-blank, erform a build that guarentees
                     # reprodicuibilty from run to run.  Cannot be used
                     # with DEBUG or TEST

TEST  =              # If non-blank, use the compiler options defined in
                     # the FFLAGS_TEST and CFLAGS_TEST macros.  Cannot be
                     # use with REPRO or DEBUG

VERBOSE =            # If non-blank, add additional verbosity compiler
                     # options

OPENMP =             # If non-blank, compile with openmp enabled

NETCDF =             # If value is '3' and CPPDEFS contains
                     # '-Duse_netCDF', then the additional cpp macro
                     # '-Duse_LARGEFILE' is added to the CPPDEFS macro.

                     # A list of -I Include directories to be added to the
                     # the compile command.
INCLUDES :=
# Optional YAML support; keep silent if pkg-config entry is missing.
YAML_CFLAGS := $(shell pkg-config --cflags yaml-0.1 2>/dev/null)
ifneq ($(strip $(YAML_CFLAGS)),)
  INCLUDES += $(YAML_CFLAGS)
endif

COVERAGE =           # Add the code coverage compile options.

USE_R4 =             # If non-blank, use R4 for reals

# Need to use at least GNU Make version 3.81
need := 3.81
ok := $(filter $(need),$(firstword $(sort $(MAKE_VERSION) $(need))))
ifneq ($(need),$(ok))
$(error Need at least make version $(need).  Load module gmake/3.81)
endif

# REPRO, DEBUG and TEST need to be mutually exclusive of each other.
# Make sure the user hasn't supplied two at the same time
ifdef REPRO
ifneq ($(DEBUG),)
$(error Options REPRO and DEBUG cannot be used together)
else ifneq ($(TEST),)
$(error Options REPRO and TEST cannot be used together)
endif
else ifdef DEBUG
ifneq ($(TEST),)
$(error Options DEBUG and TEST cannot be used together)
endif
endif

ifdef USE_R4
REAL_PRECISION := -fdefault-real-4
CPPDEFS += -DOVERLOAD_R4
else
REAL_PRECISION := -fdefault-real-8
endif

# Required Preprocessor Macros:
CPPDEFS += -Duse_netCDF

# Additional Preprocessor Macros needed due to  Autotools and CMake
CPPDEFS += -DHAVE_SCHED_GETAFFINITY -DHAVE_GETTID

# Macro for Fortran preprocessor
FPPFLAGS := $(INCLUDES)
# Fortran Compiler flags for the NetCDF library
FPPFLAGS += $(shell nf-config --fflags)

# Base set of Fortran compiler flags
FFLAGS := -g -fbacktrace -fcray-pointer -fdefault-real-8 -fdefault-double-8 \
  -Waliasing -ffree-line-length-none -fno-range-check -fallow-argument-mismatch \
  -fallow-invalid-boz

# Flags based on perforance target (production (OPT), reproduction (REPRO), or debug (DEBUG)
FFLAGS_OPT = -O2 -fno-expensive-optimizations
FFLAGS_REPRO =

# In HDF5 1.14.3, certain operations trigger floating point exceptions.
#   Until resolved, we must temporarily disable them.
FFLAGS_DEBUG = -O0 -W -fbounds-check -ffpe-trap=invalid,zero,overflow
#FFLAGS_DEBUG = -O0 -W -fbounds-check

# Flags to add additional build options
FFLAGS_OPENMP = -fopenmp
FFLAGS_VERBOSE = -Wall -Wextra
FFLAGS_COVERAGE =

# Macro for C preprocessor
CPPFLAGS := -D__IFC $(INCLUDES)
# C Compiler flags for the NetCDF library
CPPFLAGS += $(shell nc-config --cflags)

# Base set of C compiler flags
CFLAGS :=

# Flags based on perforance target (production (OPT), reproduction (REPRO), or debug (DEBUG)
CFLAGS_OPT = -O2
CFLAGS_REPRO = -O2
CFLAGS_DEBUG = -O0 -g

# Flags to add additional build options
CFLAGS_OPENMP = -fopenmp
CFLAGS_VERBOSE = -Wall -Wextra
CFLAGS_COVERAGE =

# Optional Testing compile flags.  Mutually exclusive from DEBUG, REPRO, and OPT
# *_TEST will match the production if no new option(s) is(are) to be tested.
FFLAGS_TEST := $(FFLAGS_OPT)
CFLAGS_TEST := $(CFLAGS_OPT)

# Linking flags
LDFLAGS :=
LDFLAGS_OPENMP := -fopenmp
LDFLAGS_VERBOSE :=
LDFLAGS_COVERAGE :=

# List of -L library directories to be added to the compile and linking commands
LIBS :=
YAML_LIBS := $(shell pkg-config --libs yaml-0.1 2>/dev/null)
ifneq ($(strip $(YAML_LIBS)),)
  LIBS += $(YAML_LIBS)
endif

# Manually apply netCDF library paths as RPATHs when available via pkg-config
NETCDF_RPATH := $(shell pkg-config --libs-only-L netcdf 2>/dev/null | sed 's/-L/-Wl,-rpath,/g')
LIBS += $(NETCDF_RPATH)

# Get compile flags based on target macros.
ifdef REPRO
CFLAGS += $(CFLAGS_REPRO)
FFLAGS += $(FFLAGS_REPRO)
else ifdef DEBUG
CFLAGS += $(CFLAGS_DEBUG)
FFLAGS += $(FFLAGS_DEBUG)
else ifdef TEST
CFLAGS += $(CFLAGS_TEST)
FFLAGS += $(FFLAGS_TEST)
else
CFLAGS += $(CFLAGS_OPT)
FFLAGS += $(FFLAGS_OPT)
endif

ifdef OPENMP
CFLAGS += $(CFLAGS_OPENMP)
FFLAGS += $(FFLAGS_OPENMP)
LDFLAGS += $(LDFLAGS_OPENMP)
endif

ifdef VERBOSE
CFLAGS += $(CFLAGS_VERBOSE)
FFLAGS += $(FFLAGS_VERBOSE)
LDFLAGS += $(LDFLAGS_VERBOSE)
endif

ifeq ($(NETCDF),3)
  # add the use_LARGEFILE cppdef
  CPPDEFS += -Duse_LARGEFILE
endif

ifdef COVERAGE
ifdef BUILDROOT
PROF_DIR=-prof-dir=$(BUILDROOT)
endif
CFLAGS += $(CFLAGS_COVERAGE) $(PROF_DIR)
FFLAGS += $(FFLAGS_COVERAGE) $(PROF_DIR)
LDFLAGS += $(LDFLAGS_COVERAGE) $(PROF_DIR)
endif

LDFLAGS += $(LIBS)

#---------------------------------------------------------------------------
# you should never need to change any lines below.

# see the MIPSPro F90 manual for more details on some of the file extensions
# discussed here.
# this makefile template recognizes fortran sourcefiles with extensions
# .f, .f90, .F, .F90. Given a sourcefile <file>.<ext>, where <ext> is one of
# the above, this provides a number of default actions:

# make <file>.opt       create an optimization report
# make <file>.o         create an object file
# make <file>.s         create an assembly listing
# make <file>.x         create an executable file, assuming standalone
#                       source
# make <file>.i         create a preprocessed file (for .F)
# make <file>.i90       create a preprocessed file (for .F90)

# The macro TMPFILES is provided to slate files like the above for removal.

RM = rm -f
TMPFILES = .*.m *.B *.L *.i *.i90 *.l *.s *.mod *.opt

.SUFFIXES: .F .F90 .H .L .T .f .f90 .h .i .i90 .l .o .s .opt .x

.f.L:
	$(FC) $(FFLAGS) -c -listing $*.f
.f.opt:
	$(FC) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.f
.f.l:
	$(FC) $(FFLAGS) -c $(LIST) $*.f
.f.T:
	$(FC) $(FFLAGS) -c -cif $*.f
.f.o:
	$(FC) $(FFLAGS) -c $*.f
.f.s:
	$(FC) $(FFLAGS) -S $*.f
.f.x:
	$(FC) $(FFLAGS) -o $*.x $*.f *.o $(LDFLAGS)
.f90.L:
	$(FC) $(FFLAGS) -c -listing $*.f90
.f90.opt:
	$(FC) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.f90
.f90.l:
	$(FC) $(FFLAGS) -c $(LIST) $*.f90
.f90.T:
	$(FC) $(FFLAGS) -c -cif $*.f90
.f90.o:
	$(FC) $(FFLAGS) -c $*.f90
.f90.s:
	$(FC) $(FFLAGS) -c -S $*.f90
.f90.x:
	$(FC) $(FFLAGS) -o $*.x $*.f90 *.o $(LDFLAGS)
.F.L:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -listing $*.F
.F.opt:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.F
.F.l:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $(LIST) $*.F
.F.T:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -cif $*.F
.F.f:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -EP $*.F > $*.f
.F.i:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -P $*.F
.F.o:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $*.F
.F.s:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -S $*.F
.F.x:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -o $*.x $*.F *.o $(LDFLAGS)
.F90.L:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -listing $*.F90
.F90.opt:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.F90
.F90.l:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $(LIST) $*.F90
.F90.T:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -cif $*.F90
.F90.f90:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -EP $*.F90 > $*.f90
.F90.i90:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -P $*.F90
.F90.o:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $*.F90
.F90.s:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -S $*.F90
.F90.x:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -o $*.x $*.F90 *.o $(LDFLAGS)
```