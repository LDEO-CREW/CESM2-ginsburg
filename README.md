
# CESM2.2 Ginsburg userguide      

## Basics of running the model       

1.  Creating a new case  
    Example `./create_newcase --case aquaQPC6 --compset QPC6 --res f19_f19_mg17 --mach ginsburg`  
    `--case aquaQPC6` is the name you give the run  
    `--compset` is chosen from [available sets](https://www.cesm.ucar.edu/models/cesm2/cesm/compsets.html)  
    `--res` is resolution from available choices, explained in next section  
    
2.  [Case Resolution](https://www.cesm.ucar.edu/models/cesm2/config/grids.html) (grid size)  
    Example: `f19_f19_mg17`  
    fnn, where n is the resolution  
    f19 = approximately “2-degree” finite-volume grid = 1.9 x 2.5  
    The first fnn refers to atmos resolution  
    m is the ocean mask, mg17 = ocean mask gx1v7 = 1 degree resolution version 7  
    
3.  Set up the case  
	Inside case root `./case.setup`  
    `./preview_run` before building to ensure batch was created correctly (doing this now will save time later when you ./case_submit to the batch system)  
    
4.  Build case  
    Log on interactively; `srun --pty -t 0-01:00 -A crew /bin/bash`  
    `Module load` netcdf-fortran-intel/4.5.3, netcdf/gcc/64/gcc/64/4.7.4, slurm/20.02.6, svn/1.10.6-1, python37  
    `./case.build --clean` to get rid of old builds & `./case.build --clean-all`  
    `./case.build`  
    This may take some time
    
5.  Run case  
	`./xmlchange STOP_OPTION=nmonths`  
	`./xmlchange STOP_N=3`  
    `./xmlchange JOB_WALLCLOCK_TIME=02:30:00 --subgroup case.run`  
    `./xmlchange DOUT_S=FALSE/TRUE` Only False for testing purposes, true if want output    
    `./case.submit`  to submit the batch job

6.  Run longer  
    `./xmlchange CONTINUE_RUN=TRUE`  
    Can also set RESUBMIT=# and after each run it will submit the job to the batch and continue the run longer  
    
7.  configure cam history files in [user_nl_cam]( https://www.cesm.ucar.edu/models/cesm2/settings/current/cam_nml.html)  
CAM is set up by default to output a set of fields to a single monthly average history file h0.YYYY-MM  
    Anything labeled with r, including rh,rh0,etc. Is part of the resubmit process  
     After a run, The h0 files are found in atm/hist and the r files in rest  
	
	nhtfrq is the frequency of output, by default it is 0 (monthly), you can add a history file by making a comma seperated list of frequencies (add this to user_nl_cam)  
	`nhtfrq = 0, 1, -48`   
	negative values correspond to hourly output (-48 gets data every 48 hours) and positive is time steps (only get data each time steo for debugging)  
	ndens is 1=4-byte output or 2=8-byte output, default is 2 but causes values to be odd so instead use 1, again in a comma seperated list   
	`ndens=1,1`  
	`fincl2 = 'T:I'`, this will include instantanious temperature in the second history file.  Add [more variables](https://www.cesm.ucar.edu/models/cesm2/atmosphere/docs/ug6/hist_flds_f2000.html) in a comma seperated list.
	These can be changed at anytime; to save time run a couple years with ndens=1 and then when you run for longer (section 6) and want more output add in the rest.      

## Modifications       
8.  Run with added chemistry     
    When adding chemistry, run the first day with these settings using NTASKS=-1 then continue to run normally. Without this the case might not reach equilibrium and run will fail on the first simulated day          
	fv_nsplit = 128  
	fv_nspltrac = 128  
	fv_nspltvrm = 128     
	inithist = "DAILY"    

9.  Run with user defined SSTs         
    First obtain an example prescribed SST file you can edit (create_new, setup and build any F compset)          
    The perscribed SST file will be labeled sst_... and the location of the file can be found in that compsets atm_in file (located in CaseDocs).       
    Best practice is to copy that file to your user directory         
    In a jupyter notebook edit the variable SST_cpl       
    Save only your new SST_cpl variable to a netcdf file (.nc)        
    Quick check: open your saved .nc file and make sure the SST_cpl data is the stored the same way as the example SST file.           

    To impliment the new SSTs in the model         
    create_newcase, use the long name to set the compset, use AQPFILE for DOCN, and include --run-unsupported since this is now an untested setup       
    ./create_newcase --case aquaQPC6_f09_SSTwave2  --compset 2000_CAM60_SLND_SICE_DOCN%AQPFILE_SROF_SGLC_SWAV --res f09_f09_mg17 --mach ginsburg --run-unsupported       
    First, XML change DOCN_AQP_FILENAME = "name_of_sst_file.nc"  
    Need to edit CaseDocs, to obtain CaseDocs setup the case (./case.setup) and run ./preview_namelist.        
    In CaseDocs, cp docn.streams.txt.aquapfile to CaseRoot, where other user_namelists are. rename it user_docn.streams.txt.aquapfile (can use mv command).       
    Edit the file with the name and location of the new SST file (can use emacs -nw filename).          
    re setup the case (./case.setup -r) and run ./preview_namelist again.       
    Go back into CaseDocs to check that the original docn.streams.txt.aquapfile reflects your changes.      


10. cosp (cloud observation simulator package)       
    to turn on cosp append it to the cam configure options using xml      
    ./xmlchange -append CAM_CONFIG_OPTS=-cosp        
    Another option for turning on cosp that is handy if all your cases need it on is shown in the next section         
    In order to obtain the ISCCP histogram the following also need to be in user_nl_cam, regardless of how you turn on cosp.       
    cosp_isccp = .true.      
    cosp_lisccp_sim = .true.       

11. make new case compset options        
    adding more compset options can be helpful when making new cases as you can select more about this case instead of using xml.      
    above is one way to turn on cosp, but we can also make cosp a subselection of cam that we can call when we create a new case      
    First find the configure_component file for cam which is in cesm2.2/components/cam/cime_config.      
    under the cam header I added [%COSP] to the CAM6 line because thats the physics ill be using but can be added to any of the cam versions      
    Next under the CAM Options header I added a description <desc option="COSP"       >CAM -cosp configuration option</desc>       
    Then under CAM_CONFIG_OPTS I added what the model will do when we call cam60%cosp  <value compset="CAM60%COSP">-cosp</value>       
    which is essentially the cammand line code I would have run each time had I not set this up.       
    now when we create a new case our compset option for cam CAM60 can include cosp like this CAM60%COSP       
 

11.  Troubleshooting  
    Problems with machine (CESM requirments not met, svn or other module not working)  
    Email IT (ginsburg is columbia’s machine) [hpc-support@columbia.edu](mailto:hpc-support@columbia.edu)  
    IT usually wants one question per email, this will create a ticket for each question  
 
   a.  Missing required modules  
    Create virtual environment (best is conda env)  
    This [wiki](https://hprc.tamu.edu/wiki/SW:Anaconda#Managing_Anaconda_Virtual_Environments) explains the steps  
    Might need to start with different anaconda version  
    `module avail` to see available versions  
    
    b. Problems with CESM or CIME  
	 [CESM discussion board](https://bb.cgd.ucar.edu/cesm/)  
    Create account; If no change when creating account, your username or password isnt valid. Successfully creating an account should automatically take you to your home page or to a login page.  
    
Problems with this documentation, email [nicolen@ldeo.columbia.edu](mailto:nicolen@ldeo.columbia.edu)  