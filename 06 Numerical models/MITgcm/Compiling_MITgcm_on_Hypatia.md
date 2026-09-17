### Login and Setup

1. Connect to Hypatia:
    
    ssh -Y your.email@hypatia.st-andrews.ac.uk
   (see post on connecting to Hypatia for further info) 
    
3. Configure Git on the Cluster: 
    
    Set Git identity once on the Hypatia system to ensure commits made on the cluster are tracked on Github 
    
    git config --global user.email "your.email@st-andrews.ac.uk"
    git config --global user.name "your.username"
    git config --global credential.helper cache
    
4. Clone GitHub fork of MITgcm: 
    
    Clone your personal fork to your home directory to create a working copy from which you can build and edit the model:
    
    git clone https://github.com/your.username/MITgcm.git
    cd MITgcm
    
5. Connect fork to the official MITgcm repo:
    
    Add the source repo as an upstream remote:
    
    git remote add upstream https://github.com/MITgcm/MITgcm.git
    git remote -v
    
    And fetch the latest official code
    
    git fetch upstream
    
6. Cretate a new branch from the master branch 
    
    git checkout -b unmodified upstream/master
    
    and push that branch to the GitHub fork 
    
    git push -u origin unmodified
    
    So that the Hypatia copy is linked to GitHub and has both a personal fork and an official upstream source
    

### Compiling MITgcm on Hypatia

(compiling verification/exp2) 

MITgcm is not compiles as a single universal source tree. Each experiment defines its own numerical configuration (through local header and package files in ../code). 

1. Move into chosen build directory
    
    cd ~/MITgcm/verification/exp2/build
    
2. Choose compiler option
    
    Can find which compiler option is available with which eg. which fortran which ifort
    
    ../../../tools/build_options/linux_amd64_gfortran
    
3. Generate the Makefile with the experiment specific code directory 
    
    ../../../tools/genmake2 -mods ../code -optfile ../../../tools/build_options/linux_amd64_gfortran
    
4. Build the model 
    
    Once the Makefile is generated, compile 
    
    make depend
    
    make 
    
    which produces the executable: mitgcmuv , located in:  ~/MITgcm/verification/exp2/build
    

### Running the compiled exectable

1. Move to the run directory 
    
    cd ~/MITgcm/verification/exp2/run
    
2. Link the input files and copy the executable 
    
    Link the required input data and place the executable in the run directory. 
    
    ln -s ../input/* .
    cp ../build/mitgcmuv .
    
3. Execute the model: 
    
    .mitgcmuv
    
    Sucessful output ends with PROGRAM MAIN: Exectution Ended Normally which confirms that the build and run were successful. 
    

FULL:

ssh -Y your.email@hypatia.st-andrews.ac.uk

git config --global user.email "your.email@st-andrews.ac.uk"
git config --global user.name "your.username"
git config --global credential.helper cache

git clone https://github.com/your.username/MITgcm.git
cd MITgcm
git remote add upstream https://github.com/MITgcm/MITgcm.git
git fetch upstream
git checkout -b unmodified upstream/master
git push -u origin unmodified

cd ~/MITgcm/verification/exp2/build
~/MITgcm/tools/genmake2 -rootdir ~/MITgcm -mods ../code -mpi -optfile ~/MITgcm/tools/build_options/linux_amd64_gfortran
make depend
make

cd ../run
ln -s ../input/* .
cp ../build/mitgcmuv .
./mitgcmuv
