# Compiling CEFI-regional-mom6
General ressources:
- [CEFI-regional-mom6 documentation](https://cefi-regional-mom6.readthedocs.io/en/latest/)
- [MOM6 documentation](https://mom6.readthedocs.io/en/main/)

## Compiling on a local Ubuntu machine
Follow the instructions given in the [CEFI-regional-MOM6 Users Guide](https://cefi-regional-mom6.readthedocs.io/en/latest/BuildMOM6.html). This guide is generally comprehensive. Below are minor departures from the guide I found necessary to get the model up and running of an Ubuntu 24.04.1 LTS machine.

### 'mkmf' template
Under point 2.3 you are asked to choose a .mk file. Use the “Jammy” ubuntu file: “linux-ubuntu-jammy-gnu.mk”. This file can be found in [MOM6-examples](https://github.com/mom-ocean/mkmf/tree/c87e743a5c396d93db7dc69ad4ddd5e62d383236/templates)

Then build the model following the instructions in the guide.

### Locating the executable
The last instruction under point 2.3 suggests the executable should be located here: 
```
builds/build/YOUR_MACHINE_DIRECTORY-NAME_OF_YOUR_mk_FILE/ocean_ice/repro/MOM6SIS2
```

In my case, the 
```
YOUR_MACHINE_DIRECTORY-NAME_OF_YOUR_mk_FILE 
```
was created as two directories where
```
-NAME_OF_YOUR_mk_FILE
```
was a sub-directory to
```
YOUR_MACHINE_DIRECTORY
```
In order to reach the executable, I needed to remove the hypthen. The executable was located here:
```
builds/build/YOUR_MACHINE_DIRECTORY/NAME_OF_YOUR_mk_FILE/ocean_ice/repro/MOM6SIS2 
```

## Compiling on ARCHER2
Follow the instructions given in the [CEFI-regional-MOM6 Users Guide](https://cefi-regional-mom6.readthedocs.io/en/latest/BuildMOM6.html). Specific modifications needed for compilation on Archer2 (compilation environment and `.mk` file edits) are described [here](https://odyn-standrews.github.io/wiki/compiling-mom6-on-archer2/)

## Running the test experiment
You should now be able to run the example experiment following the guide. As an additional note, make sure to unzip the model input files into `exps/`, and check the `OceanBGC_dataset` file is located in the folder called `OceanBGC_dataset` and the `OM4_025.JRA.single_column.tar.gz` files are in the folder called `OM4_025.JRA.single_column`. Finally, go into the `exps/OM4.single_column.COBALT/INPUT` to make sure that all the soft links are working properly.

## Compiling on Hypatia

Follow the standard instructions, but create your own `.env` and `.mk` files as follows:

### `.env` file
```bash
module purge
module load gnu/14.2.0
module load openmpi/5.0.7
module load netcdf-fortran/test-4.6.2

export FC=mpifort
export CC=mpicc
export LD=mpifort
```

### `.mk` file
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

LIBS += $(shell nf-config --flibs)

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
