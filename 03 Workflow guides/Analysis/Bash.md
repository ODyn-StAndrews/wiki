# Bash

Bash is a command-line language that allows us to interact directly with a machine's operating system. There is an ocean of online resources to understand the basics (and not-so-basics) of Bash, starting on the [GNU Bash website ](https://www.gnu.org/software/bash/). Here, we collect some information and examples relevant to our work.

# Useful commands
A collection of particularly useful (and perhaps non-standard) commands that can be used at the command line or within shell scripts.

# Bash scripting for computations
## Why Bash scripting? 
Bash scripting can be an efficient tool in order to manage series of computations for different sets of input parameters, all of that on a distant machine which 99% of the time will run on Linux. 

Bash scripts are plain text files containing sequences of bash commands to be executed by the machine, and thus do not need compiling or any other program than bash.

## What can Bash do?

Plenty of things, among which file automatic filling, complex pre-calculations, managing all this through for and while loops, conditional statements, etc.

## How to use it to manage my computations?

Basically, Bash scripts can be used to automatically generate running directories, put in there all the required input files, and submit the job to e.g. SLURM.

Here is an example using a bash shell script to write and change a python file before runnning it: [[Managing computations with BASH]]