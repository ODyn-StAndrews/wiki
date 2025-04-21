[ARCHER2 User Guide](https://docs.archer2.ac.uk/user-guide/)  
[SAFE Account Portal](safe.epcc.ed.ac.uk)

# ARCHER2
ARCHER2 is the UK research community supercomputing facility. The purpose of this note is to collect relevant information for accessing and working on this facility, specific to my needs and those of my group. There is a lot of general information available on the ARCHER2 website (links below).

** If just getting started, follow the Quickstart for Users documentation (see below) **

## Useful links
Archer homepage : [https://www.archer2.ac.uk/](https://www.archer2.ac.uk/)  

Documentation homepage : [https://docs.archer2.ac.uk/](https://docs.archer2.ac.uk/)  

Quickstart guides : 
- Getting an account, requesting access to projects, logging on, basic filesystem details: [https://docs.archer2.ac.uk/quick-start/quickstart-users/](https://docs.archer2.ac.uk/quick-start/quickstart-users/)
- Compiling and running software : [https://docs.archer2.ac.uk/quick-start/quickstart-developers/](https://docs.archer2.ac.uk/quick-start/quickstart-users/)

User and Best Practice guide : [https://docs.archer2.ac.uk/user-guide/](https://docs.archer2.ac.uk/user-guide/)

## Organization
Adminisitrative details (e.g. accounts, projects, allocations) are handled via SAFE ([safe.epcc.ed.ac.uk](safe.epcc.ed.ac.uk)). Users need to sign up for an account on there, and through there, request access to a specific project (see project details below).

SAFE is also the place in which a project PI can approve users, check usage, and observe allocation.

## Access and login
Example `.ssh/config` entry
```
Host archer2
        Hostname login.archer2.ac.uk
        User graemem
        IdentityFile ~/.ssh/id_rsa_archer
```
Note that your `ssh` key has to have a password associated with it (in addition to your ARCHER2 (LDAP) password).

## Directory structure
User directories are organized by projects, with space available in both `/work/project_id/project_id/$USER` and `/home/project_id/project_id/$USER`. The `work` directory has more storage space.

Within each project `work` directory there is a `shared` folder, which all user can read from and write to. If you wish to share something with users not in the project (e.g. ARCHER2 admin for debugging), you need to put it in `/work/project_id/shared`.

## Project details
### CO2 and climate change: high-latitude oceans 
- Project ID : **e786**
- PI username/email : **gam24@st-andrews.ac.uk**

## Analysis and running jobs
Details on partitions and slurm scheduler [here](https://docs.archer2.ac.uk/user-guide/scheduler).

Command for starting an interactive session on the data-viz partition (`serial`):
```
srun --nodes=1 --time=01:30:00 --account=e786 --partition=serial --qos=serial --pty /bin/bash
```

## Software
Supporting documentation for using a wide array of Research Software on `archer2` (including MITgcm, NEMO, and CESM2) is provided [here](https://docs.archer2.ac.uk/research-software/).

### Python, virtual environments, and package managers
General documentation on using python is included [here](https://docs.archer2.ac.uk/user-guide/python/).

For installing your own packages, you are recommended to use `pip` (documentation [here](https://docs.archer2.ac.uk/user-guide/python/#installing-your-own-python-packages-with-pip)) rather than `conda`, especially when hoping to use those packages on the compute nodes. You can do this in a virtual environment (recommended) using the `venv` software (follow the instructions at the link above).

**Notes** [Graeme]
I  have taken the approach of using `pip` to install modules that I hope to use on the compute nodes, e.g. for any large computation. This causes problems when certain software (e.g. OceanParcels) are configured quite specifically to be installed with `conda`. I have not yet worked around this.

I have also installed an instance of `miniconda` in my home directory, allowing me to install and use packages on the login nodes and data visualisation nodes (`qos=serial`). 

### Jupyter Lab
Jupyter lab can be run either from the login/data vis nodes, or from the compute nodes, following the procedure outlined [here](https://docs.archer2.ac.uk/user-guide/python/#using-jupyterlab-on-archer2). The only addition made to this procedure is to select a port number (instead of the default `8888`). Using the default can lead to errors if your local machine is already using that port.

The command to launch Jupyter Lab is:
```
jupyter lab --no-browser --ip=0.0.0.0 --port=8659
```
The `ip` flag is required; the `port` number can be whatever you want (within a certain range).

## Additonal information
### Acknowledgment
From the ARCHER2 website:
> You should use the following phrase to acknowledge ARCHER2 for all research outputs that were generated using the ARCHER2 service:
> 
> This work used the ARCHER2 UK National Supercomputing Service (https://www.archer2.ac.uk).
> 
> You should also tag outputs with the keyword "ARCHER2" whenever possible.