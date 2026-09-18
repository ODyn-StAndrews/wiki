# Hypatia

Hypatia is the St Andrews HPC (see details here [https://www.st-andrews.ac.uk/high-performance-computing/](https://www.st-andrews.ac.uk/high-performance-computing/)) These are some instructions on how to connect to Hypatia, run Jupyter Notebooks, and how to submit batch scripts, largely based off the notes by Simon Lee: https://simonleewx.com/hypatia-tips-and-tricks/

## Getting started
You need a Hypatia log-in -- if you don't have one, follow the link to the St Andrews HPC information above and request one, or email Herbert Fruchtl (Herbert.fruchtl@st-andrews.ac.uk).

### Logging in
SSH into the Hypatia login node, using your username in place of fg77 (do not run large operations on the login node)
`ssh -Y fg77@hypatia.st-andrews.ac.uk`

You will be asked to enter your password. You can use passkeys so you don't have to do this every time.

### Installing conda

`install-conda`

`conda create <ENV-NAME>`

Then activate the environment
`conda activate<ENV-NAME>`

### Storage
Storage in the home directory (e.g. `/home/<username>/`) is backed up. Scratch storage can be found at `/sharedscratch/<username>`.

## Running Jupyter Notebooks in Browser
(Taken from https://simonleewx.com/hypatia-tips-and-tricks/)
To use Jupyter notebooks on Hypatia, you need to request compute, set up a server, and then connect to it.
Create two files:

```
#!/bin/bash  
#Run this on the login node to acquire compute node resources and return interactive shell  
#Usage: ./run_srun.sh <time> <memory> <partition>  
# Example (1 hour 24GB on small-short): ./run_srun.sh 0-1:00 24G small-short  
**TIME**=$1  
**MEMORY**=$2  
**PARTITION**=$3  
srun --pty -t "$TIME" -p "$PARTITION" --mem="$MEMORY" /bin/bash  
```
```
#!/bin/bash  
# Run this on compute node to launch JupyterLab  
source ~/.bashrc  
conda activate <ENV_NAME>  
**hostvar**=$(hostname -i)  
echo $hostvar  
jupyter-lab --no-browser --ip=$hostvar  
```
Ensure these are executable using `chmod +x`

When you log into Hypatia, run: 
`./run_srun.sh 0-1:00 24G small-short`
`./launch_jl.sh`

It will print out something like: 
`172.22.10.XX`

Then open a new tab in the terminal and enter:
`ssh -L 8080:172.22.10.XX:8888 fg77@hypatia.st-andrews.ac.uk`
with the correct printed out numbers. Sometimes the :8888 will be an :8889, so watch out!

Now in the browser, navigate to localhost:8080

The first time, you'll probably need to enter a token which can be found where you set up the server in the first terminal window.

You can now use Jupyter notebooks in browser.

## Example Batch Scripts
For running big jobs, you can use slurm to submit scripts to one of several partitions. (See here for more information https://www.st-andrews.ac.uk/high-performance-computing/help-and-contact/using/)

Here's an example script to submit a file called test.py. You will likely need to change details to work with your jobs.
Create a file called `submit_script.slurm` (or whatever), and paste in:
```
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --time=12:00:00
#SBATCH --mem=64GB
#SBATCH --partition=large-short
#SBATCH --nodes=1
#SBATCH --error='logs/test_job%j.txt'

source ~/.bashrc

conda activate <env_name>

python /home/<username>/test.py
```

To submit this job, do:
`sbatch submit_script.slurm`

To check on the status of the job, do:
`squeue -u <username>`

To cancel the job, do:
`scancel -u <job_id>`
