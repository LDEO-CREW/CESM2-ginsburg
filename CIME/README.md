
# Downloading & Porting  

This document is for development purposes only. You do not need to repeat these steps to use CESM2.2 on Ginsburg.  

### A preliminary step is to create an output directory on your machine [(Ginsburg)](https://confluence.columbia.edu/confluence/display/rcs/Ginsburg+HPC+Cluster+User+Documentation)  

   1. `mkdir /burg/crew/users/<UNI>`  
   2. Exchange crew with your groups account on ginsburg  

### [Downloading CESM2.2](https://www.cesm.ucar.edu/models/cesm2/release_download.html)

   1. Check machine meets [CESM2.2 requirements](https://escomp.github.io/CESM/versions/cesm2.2/html/introduction.html#cesm2-software-operating-system-prerequisites)  
       1.1. Most package versions can be checked with `module avail`  
       1.2. `perl -v`  
       1.3. If using the intel compiler *Intel-parallel-studio/2020*, you must also use a fortran netcdf built by that compiler, `module show <netcdf_name>`  

   2. Git a clone of the model  
       2.1. Chose a name and location for the directory containing the model (CESM2.2). This is seperate from output directory (/burg/crew/projects)  
       2.2. Run the first line from [this document](https://escomp.github.io/CESM/versions/cesm2.2/html/downloading_cesm.html#downloading-the-code-and-scripts), exchanging _my_cesm_sandbox_ for your chosen file name (CESM2.2)  
  
   3. Check out individual model components  
       3.1. Uses svn so easier to do interactively  
       3.2. Log on to node Interactivly using slurm `srun --pty -t 0-01:00 -A crew /bin/bash`  
          Change crew to your groups account name  
          -t is time and 0-01:00 is 0 days, 1 hour, and 0 mins  
       3.3. Change directory into your named directory (CESM2.2) `cd CESM2.2`  
       3.4. Run a script that checks out the components for you `./manage_externals/checkout_externals`  
       3.5. Confirm successful download `./manage_externals/checkout_externals -S`  
          Should not see any e-  
          If you do see e-, first try `svn ls https://svn-ccsm-models.cgd.ucar.edu/ww3/release_tags` and recheckout the components  
          e-o is okay, those are optional  
  
### Port Cime on Ginsburg

   1. Need to define machine, compiler, and batch jobs. I defined gisburg in config_machine, config_compiler, and config_batch (included in this repo)  
