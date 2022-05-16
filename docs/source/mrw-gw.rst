Using UFS Medium-Range Weather Application with Global Workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set-up
^^^^^^

1. Clone repository and checkout global workflow::

      git clone https://github.com/ufs-community/ufs-mrweather-app.git
      cd ufs-mrweather-app
      ./manage_externals/checkout_externals

2. Check if checkout of externals was successful using::

      ./manage_externals/checkout_externals -S

3. Build UFS model and global-workflow components::

      sh build_global-workflow.sh [-c]
      (ONLY use the -c option to compile for coupled UFS; requires different physics packages and APP argument when running
      setup_expt.py in step 5. )

4. Create COMROT and EXPDIR directories. The experiment and workflow set-up scripts in following steps will point to these paths (EXPDIR will contain model configuration files, Rocoto/Cron files, Rocoto/SLURM logs; COMROT will contain experiment outputs & GFS model logs). Initial conditions also need to be staged in COMROT (step 6).

5. Run experiment generator script::

      cd ufs-mrweather-app/global-workflow/ush/rocoto
      ./setup_expt.py forecast-only --pslot $EXP_NAME --idate $YYYYMMDDCC --edate $YYYYMMDDCC --resdet $MODEL_RESOLUTION --comrot $PATH_TO_YOUR_COMROT_DIR --expdir $PATH_TO_YOUR_EXPDIR

      (example with COMROT and EXPDIR paths)::

      ./setup_expt.py forecast-only --pslot test --idate 2020010118 --edate 2020010118 --resdet 384 --comrot /work/noaa/stmp/cbook/COMROT --expdir /work/noaa/epic-ps/cbook/uncoupled/EXPDIR
  
  Note that idate=edate, which seems to be required for free-forecast mode to run properly without attempting to invoke DA components of the workflow.

6. Copy IC files into COMROT/$PSLOT, where $PSLOT is the name of the experiment-specific subdirectory created in step 5. IC directory name should be like::
     
      gfs.20220101, with structure: gfs.$YYYYMMDD/CC/atmos. INPUT folder within …/atmos/ contains IC files needed for GFS ATM to run.

7. Edit config.base in $EXPDIR/$PSLOT (ACCOUNT, HOMEDIR, STMP/PTMP, FHMAX). ACCOUNT should correspond to a project in which you are able to submit/run jobs (e.g.: marine-cpu, fv3-cpu), HOMEDIR is a 'no-scrub' path where archive files will be placed, STMP/PTMP are large-disk-space areas where run files (RUNDIRS) are temporarily located, but scrubbed after experiment completion. FHMAX_GFS_[OO, 06, 12, 18] sets the maximum number of hours for the free-forecast to run (default is 384.); to run, for example, the forecast model for two days, change to 48.

8. Run ./setup_workflow_fcstonly.py --expdir $EXPDIR/$PSLOT

   This will generate crontab and xml files for the experiment in $EXPDIR/$PSLOT.

9.  Submit job through crontab by copying entry in $PSLOT.crontab into crontab via crontab -e or by doing::

      crontab $PSLOT.crontab

10. Monitor status of workflow using rocotostat::
      
      rocotostat -d /path/to/workflow/database/file -w /path/to/workflow/xml/file [-c YYYYMMDDCCmm,[YYYYMMDDCCmm,...]] [-t taskname,[taskname,...]] [-s] [-T]
      e.g.: rocotostat -d $PSLOT.db -w $PSLOT.xml

11. Check status of specific task/job::
      
      rocotocheck -d /path/to/workflow/database/file -w /path/to/workflow/xml/file -c YYYYMMDDCCmm -t taskname

    Helpful documentation:
    https://github.com/NOAA-EMC/global-workflow/wiki/Run-Global-Workflow

Initial Conditions
^^^^^^^^^^^^^^^^^^

The FV3GFS component of the global workflow requires GFS intitial conditions at the appropriate date/time and resolution/layer number to intialize the FV3 model for a given experiment window. GFS initial conditions are available for retrival through HPSS on Hera, but must be staged on Orion and other machines without HPSS access. Operational GFS intitial conditions are in C768 resolution, thus for lower-resolution experiments, the initial condition files must be remapped into the desired resolution. The retrieval and conversion of initial conditions can be done automatically on Hera during the 'getic' and 'gfs_init' subtasks, but must be staged and converted into FV3 tile format (both for C768 and lower resolution tests) on Orion and other machines. The UFS_UTIL tool "chgres" can be implemented to handle the conversion from raw GFS intial condition files into the FV3 format (instructions here:https://github.com/ulmononian/EPIC-Documentation/blob/main/docs/source/ufs_utils.rst). Once converted into the desired resolution in FV3 format, initial conditions should be staged in the experment-specific subdirectory of the user's COMROT. 

Example initial conditons for 2018 Hurricane Michael (all resolutions) are available here: /work/noaa/epic-ps/ufs-mrw-v2.0/mrw_test_cases/ICs.


Test Cases
^^^^^^^^^^

A suite of tests were conducted corresponding to the 2018 Hurricane Michael UFS Weather Model test case (https://ufs-case-studies.readthedocs.io/en/develop/2018Michael.html). The tests were initialized at 00z October 7, 2018 and ran for a 24-hour forecast length. Tests were performed at all resolutions (C768, C384, C192, C96, C48). Test outputs and configurations are located here: /work/noaa/epic-ps/ufs-mrw-v2.0/mrw_test_cases.



