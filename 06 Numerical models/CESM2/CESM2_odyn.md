# CESM2 Setup Guide for Hypatia

*Contributors*: Freya Gibbon

Based on the notes of Anna Mackie (arm33@st-andrews.ac.uk)

This will get you set up able to run CESM2 cases on Hypatia. For more information on CESM2, see the quickstart guide: https://escomp.github.io/CESM/release-cesm2/index.html

  

### Log into Hypatia

- If outside the University wifi, make sure you’re connected to the VPN (Cisco Secure Client)
- In the terminal: (`<uun>` is your username e.g. fg77)
	`ssh<uun>@hypatia.st-andrews.ac.uk`
- Enter password
- Make a symbolic link to the shared scratch
	`ln -s /sharedscratch/$USER ~/work`

  

### Clone CESM2
- In your home directory, do
	` git clone -b release-cesm2.1.5 https://github.com/ESCOMP/CESM.git my_cesm_sandbox`
	` cd my cesm_sandbox`
	` ./manage_externals/checkout_externals` (do this one twice)

### Install conda in the conda directory
- Go to the condo directory:
	` cd /gpfs01/software/conda/ `
	` mkdir <uun> `
	`cd <uun>`
	`install-conda`
- Then log out and log back in again

### Set up the CESM conda environment

`cd ~/my_cesm_sandbox`
`conda create -n cesm_36_1 python=3.6 perl`
`conda activate cesm_36_1`
`conda install -c bioconda perl-xml-libxml`
`conda install -c conda-forge esmf`

### Copy across these files (specific for Hypatia)

`mkdir ~/.cime`
`cp /sharedscratch/mpb20/for_anna/config_machines.xml ~/.cime/`
`cp /sharedscratch/mpb20/for_anna/config_compilers.xml ~/.cime/`
`cp /sharedscratch/mpb20/for_anna/config_batch.xml ~/.cime/`
`cp /sharedscratch/mpb20/for_anna/setup_for_cesm.sh ~/.cime/`
`cp /sharedscratch/mpb20/for_anna/xmlcheck.sh ~/.cime/`
`cd ~/.cime`
- Now edit the file setup_for_cesm.sh for your paths; i.e., change mpb20 for your `<uun>`

  ## Running an experiment
Now you are ready to create a case! This is how to set up an experiment in CESM2. 

The above file structure is slightly different to the one in the CESM2 tutorial so be aware of that when trying to find your case directory (i.e. your case directories will be in ~/.cime, not ~/cesm/my_cesm_sandbox/cime)

The following is how to create a case using the F2000_c4 compset (see more information about CESM2 component sets here: https://escomp.github.io/CESM/release-cesm2/cesm_configurations.html#cesm2-component-sets)

`conda activate cesm_36_1`
`cd ~/.cime`
`. setup_for_cesm.sh`
`bash xmlcheck.sh`
`${CIMEROOT}/scripts/create_newcase --case F2000_c4 --compset F2000climo --res f19_f19_mg17`

- ${CIMEROOT}/scripts/create_newcase is the script which creates the case

- --case F2000_c4 is the case name, and the name of the case directory which will be created

- --compset F2000climo is the name of the component set you want to run

- --res f19_f19_mg17 is the resolution you want to run

- You can add more arguments here
- Now go into the case directory and run these commands.
	`cd F2000_c4`
	`./case.setup`
	` ./case.build `
- (This is the stage where you can edit things about the run using xml files or namelists. E.g. `./xmlquery STOP_N` tells you how many days your simulation is running for. Change this using e.g. `./xmlchange STOP_N = 10`)
`./case.submit`
- Well done! You have run CESM2

- Check the status of your job with `squeue -u <uun>`

- The data is archived in `~/work/CESM_outputs/archive/`

- Restart files are in `~/work/CESM_outputs/<casename>/run`
