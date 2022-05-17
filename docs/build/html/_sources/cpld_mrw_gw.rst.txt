Using UFS Medium-Range Weather Application with Global Workflow for Coupled Experiments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set-up
^^^^^^

1. Clone repository and checkout global workflow::

      git clone https://github.com/ufs-community/ufs-mrweather-app.git
      cd ufs-mrweather-app
      ./manage_externals/checkout_externals

2. Check if checkout of externals was successful using::

      ./manage_externals/checkout_externals -S

3. Build UFS model and global-workflow components::

      sh build_global-workflow.sh -c 

4. Create COMROT and EXPDIR directories. The experiment and workflow set-up scripts in following steps will point to these paths (EXPDIR will contain model configuration files, Rocoto/Cron files, Rocoto/SLURM logs; COMROT will contain experiment outputs & GFS model logs). Initial conditions will be pulled from a central location and temporarily staged in the experiment-specific subdirectory within COMROT.

5. Run the experiment set-up Python script::

        cd ush/rocoto

        ./setup_expt.py forecast-only --idate $IDATE --edate $EDATE [--app $APP] [--aerosols] [--start $START] [--gfs_cyc $GFS_CYC] [--resdet $RESDET] [--pslot $PSLOT] [--configdir $CONFIGDIR] [--comrot $COMROT] [--expdir $EXPDIR] [--icsdir $ICSDIR]

   where::
        *forecast-only is the first positional argument that instructs the setup script to produce an experiment directory for forecast only experiments.
        *$APP is the target application, one of:
        *ATM: atmosphere-only [default]
        *ATMW: atm-wave
        *S2S: atm-ocean-ice
        *S2SW: atm-ocean-ice-wave
        *--aerosols, if set, will turn on the aerosol model (GOCART)
        *$START is the start type (warm or cold [default])
        *$IDATE is the initial start date of your run (first cycle CDATE, YYYYMMDDCC)*
        *$EDATE is the ending date of your run (YYYYMMDDCC) and is the last cycle that will complete*
        *$PSLOT is the name of your experiment [default: test]
        *$CONFIGDIR is the path to the /config folder under the copy of the system you're using [default: $TOP_OF_CLONE/parm/config/]
        *$RESDET is the FV3 resolution (i.e. 768 for C768) [default: 384]
        *$GFS_CYC is the forecast frequency (0 = none, 1 = 00z only [default], 2 = 00z & 12z, 4 = all cycles)
        *$COMROT is the path to your experiment output directory. DO NOT include PSLOT folder at end of path, it’ll be built for you. [default: $HOME]
        *$EXPDIR is the path to your experiment directory where your configs will be placed and where you will find your workflow monitoring files (i.e. rocoto database and xml file). DO NOT include PSLOT folder at end of path, it will be built for you. [default: $HOME]
        *$ICSDIR is the path to the initial conditions. This is handled differently depending on whether $APP is S2S or not. If $APP is ATM or ATMW, this setting is currently ignored. If $APP is S2S or S2SW, ICs are copied from the central location to this location and the argument is required. Central locations of ICs on Hera and Orion can be found in ufs-mrweather-app/global-workflow/parm/config. 

Note that $IDATE should equal $EDATE for forecast-only experiments.

$GFS_CYC, $CONFIGDIR, and --aerosols can be removed when invoking ./setup_expt.py, as the default $GFS_CYC is one (necessary for forecast-only experiments), the default $CONFIGDIR ($TOP_OF_CLONE/parm/config) need only be changed if the user wishes to use non-standard configurations, and --aerosols only needs to be included if the user wishes to include the GOCART aerosol model component (currently experiencing issues).

Edit config.base in $EXPDIR/$PSLOT (ACCOUNT, HOMEDIR, STMP/PTMP, FHMAX). ACCOUNT should correspond to a project in which you are able to submit/run jobs (e.g.: marine-cpu, fv3-cpu), HOMEDIR is a ‘no-scrub’ path where archive files will be placed, STMP/PTMP are large-disk-space areas where run files (RUNDIRS) are temporarily located, but scrubbed after experiment completion. FHMAX_GFS_[OO, 06, 12, 18] sets the maximum number of hours for the free-forecast to run (default is 384.); to run, for example, the forecast model for two days, change to 48.

Example test
^^^^^^^^^^^^

A cold-start, forecast-only S2SW (atmosphere-ocean-ice-wave) experiment at C384 beginning on 2013100100 (without aerosols) was run for 24 hours on Orion. The experiment path is: /work/noaa/epic-ps/ufs-mrw-v2.0/coupled/EXPDIR/C384/2013100100_test. On Orion, ICs are pulled automatically from /work/noaa/global/wkolczyn/noscrub/global-workflow/IC. Note that only a limited number of ICs are available at this location. To test other dates, users need to locate and stage the appropriate atmosphere, ocean, sea ice, and wave ICs. The command to generate the experiment was::

./setup_expt.py forecast-only --pslot 2013100100_test --idate 2013100100 --edate 2013100100 --app S2SW --resdet 384 --start cold --comrot /work/noaa/epic-ps/ufs-mrw-v2.0/coupled/COMROT --expdir /work/noaa/epic-ps/ufs-mrw-v2.0/coupled/EXPDIR/C384 --icsdir  /work/noaa/epic-ps/ufs-mrw-v2.0/coupled/ICSDIR

