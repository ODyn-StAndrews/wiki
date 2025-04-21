This documentation provides useful points of reference for working on JASMIN. It *should not* be used as a step-by-step guide for working on that service. For that, there is extensive documentation on the JASMIN website.

[JASMIN help documents](https://help.jasmin.ac.uk/)  
[JASMIN Accounts Portal](https://accounts.jasmin.ac.uk/)

## Getting started
Follow the step-by-step instructions on the JASMIN [Getting Started](https://help.jasmin.ac.uk/docs/getting-started/) help page. The important steps are to generate an account, generate an `ssh` key-pair, and logging in with `ssh`.

## Logging in
Logging into JASMIN is via the `login` nodes: `login-01.jasmin.ac.uk`.

From either of the `login` nodes, you then log in to the science nodes (`sci1.jasmin.ac.uk`, `sci2.jasmin.ac.uk`, etc.) to do analysis.
### SSH config
Create a key pair following the instructions [here](https://help.jasmin.ac.uk/article/185-generate-ssh-key-pair).

Load your private key into the `ssh-agent` following the instructions [here](https://help.jasmin.ac.uk/article/187-login) under "Preparing your Credentials".

Add a `jasmin` host to your `ssh` config file, located at `~/.ssh/config` to make logging in very straightforward.
```
Host jasmin
        Hostname login1.jasmin.ac.uk
        User graemem
        IdentityFile ~/.ssh/id_rsa_jasmin
        ForwardAgent yes
```

`ForwardAgent` ensures that your SSH key is forwarded to allow access to the science machines from the login machine.

Logging in is now as simple as `ssh jasmin`.

Note that to login to the science nodes (e.g. `sci1.jasmin.ac.uk`) you need to make sure that you have forwarded the SSH key. In the above approach, this requires that the key is "added" via the `ssh-agent` protocol prior to your initial login, so that it is available to be forwarded. When you add a key, it will stay on there for some time. However, if you have not been on JASMIN in a while, you may need to repeat the "Preparing your credentials" steps prior to logging in.

It is also helpful to forward your SSH key *again* when you log on to the science nodes. I do this by again creating a `Host` in the `config` file (this time on the JASMIN login node). That config file looks like:
```
Host sci1
        Hostname sci1.jasmin.ac.uk
        ForwardAgent yes
```
Similar lines could be added for `sci2`, `sci3`, etc. Note that logging onto the science nodes with agent forwarding on is essential for interacting with GitHub.

Or you can use a proxyjump to login to the science nodes directly: 
```
Host sci1.jasmin
  Hostname sci1.jasmin.ac.uk
  User xtiliang
  IdentityFile ~/.ssh/id_rsa_jasmin
  ProxyJump jasmin
  ForwardAgent yes
  ForwardX11 yes
```
Notice you also need to add `ForwardX11 yes` to the end of jasmin config paragraph to enable the jump. Now `ssh sci1.jasmin` will login to the science mode in one step.

## Jupyter notebooks
JASMIN has a JupyterHub, making access to Jupyter Lab exceptionally straightforward. Simply navigate to [notebooks.jasmin.ac.uk](https://notebooks.jasmin.ac.uk), login, complete 2-factor authentication, and you're in. Notebooks are initiated with 4 CPUs and 16GB of RAM. This cannot be changed.

JASMIN notebooks service [documentation](https://help.jasmin.ac.uk/article/4851-jasmin-notebook-service).

To install and use different packages on the notebook service, see the documentation [here](https://help.jasmin.ac.uk/docs/interactive-computing/creating-a-virtual-environment-in-the-notebooks-service/). The recommended approach there is a bit convoluted. If you already have `conda` environments on the scientific analysis nodes (e.g. `sci1`) then you can simply install `ipykernel` to those environments, then run 
```
conda run --name name-insert-here python -m ipykernel install --user --name name-insert-here
```
from within the notebook service terminal environment, where `name-insert-here` is the name of your conda environment.

### Dask gateway to submit LOTUS job from a notebook
Sometimes, the notebook allocation (4 CPUs, 16GB) may prove insufficient for the computations required. In that case, you can start LOTUS jobs (see below for details on the LOTUS batch computing cluster) from within the Jupyter notebook.

Details on the [dask gateway service](https://github.com/cedadev/jasmin-daskgateway). First step is to apply for access to the dask gateway service on JASMIN: do so [here](https://accounts.jasmin.ac.uk/services/additional_services/dask/).

## Storing data
See [here](https://help.jasmin.ac.uk/article/176-storage#where-to-write-data) for extensive documentation on JASMIN storage.
### Personal storage
- Small files (e.g. Jupyter notebooks, figures, etc.) should be stored in your `/home` directory. This filesystem is backed up.
    - You should *not* store model output in the `/home` directory, as this will clog it up quickly.
    - A helpful command: `pdu -sh /home/users/<username>` will print your current memory usage in your home directory.
- Model output data can be temporarily stored in the `/work/scratch-nopw` filesystem. See the section **"How to use the temporary disk space"** at the link above.
    - Note that, as stated in the documentation, you should add a subdirectory based on your JASMIN username within the `scratch` directory (`mkdir /work/scratch-nopw/$USER`) and store everything in there.
### Group workspace
A Group Workspace, with 10Tb of rapid-access storage and 100Tb of tape storage is available. The name of the workspace is `co2clim`, and the path is `/gws/nopw/j04/co2clim`.  Details on accessing and working with Group Workspaces is [here](https://help.jasmin.ac.uk/article/199-introduction-to-group-workspaces).

Data can be stored on the rapid-access storage while it is being actively used. Once a project is completed, or has become stagnant, data should be transferred to tape.

## Available data
There is a wealth of data (such as CMIP, reanalysis, etc.) already stored on JASMIN, in the [CEDA archive](https://help.jasmin.ac.uk/docs/long-term-archive-storage/ceda-archive/). To get access you need to [get a CEDA account](https://help.ceda.ac.uk/article/39-ceda-account) and [link your CEDA and JASMIN accounts](https://help.ceda.ac.uk/article/5105-linking-your-jasmin-account).

Available datasets can be searched for in the [catalogue](https://help.ceda.ac.uk/article/137-ceda-data-catalogue). There is also an extremely useful [web archive](https://data.ceda.ac.uk/) giving the full file structure available on JASMIN.

## Transferring data
Help page for JASMIN data transfers can be found [here](https://help.jasmin.ac.uk/docs/data-transfer/).

### Globus
Instructions for [Command Line Interface](https://help.jasmin.ac.uk/article/4480-data-transfer-tools-globus-command-line-interface)  
Instructions for [Web Interface](https://help.jasmin.ac.uk/article/5008-data-transfer-tools-using-the-globus-web-interface)  
The relevant Globus collection is `Jasmin default collection`, and you verify permissions when attempting to access on Globus.  See [[Globus]] for further details.

### Transfers to/from ARCHER2
Procedures for transferring between ARCHER2 and JASMIN is outlined [here](https://help.jasmin.ac.uk/docs/data-transfer/transfers-from-archer2/). However, this has not been updated to accommodate the generally preferred approach, which is to use the [[Globus]] Web Interface. Both ARCHER2 and JASMIN have Globus collections, so transfers between them is straightforward. See [[Globus]] for details of how to use the web interface to perform these transfers.

### Making data available to external collaborators
On the Group Workspace, there is a `public` folder. Anything in this folder can be accessed via an HTTP protocol via the server [https://gws-access.jasmin.ac.uk/](https://gws-access.jasmin.ac.uk/). So, for example, an external collaborator could download files from that folder using `wget`:
```
wget https://gws-access.jasmin.ac.uk/public/co2clim/path/to/file .
```

## Software
Many software packages are maintained in the [Jaspy Software Environment](https://help.jasmin.ac.uk/docs/software-on-jasmin/jaspy-envs). This is a (set of) master conda environments that contain many of the packages you may use on a daily basis (xarray, xhistogram, mayplotlib, cartopy, etc.).

If there are packages or environments that you need that are not included as part of `Jaspy`, you can also install `miniconda` using the protocol outlined [here](https://help.jasmin.ac.uk/docs/software-on-jasmin/creating-and-using-miniconda-environments/).