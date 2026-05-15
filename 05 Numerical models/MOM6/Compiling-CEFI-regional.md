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


   
