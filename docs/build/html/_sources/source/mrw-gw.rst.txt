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
      setup_expt.py in step 4. )

4. Create a COMROT and EXPDIR. The experiment and workflow set-up scripts in following steps will point to these paths. Initial conditions will also need to be placed in COMROT.

5. Run experiment generator script::

      cd ufs-mrweather-app/global-workflow/ush/rocoto
      ./setup_expt.py forecast-only --pslot $EXP_NAME --idate 2020010100 --edate 2020010118 --resdet 384 --gfs_cyc 4 --comrot $PATH_TO_YOUR_COMROT_DIR --expdir $PATH_TO_YOUR_EXPDIR

      (example with COMROT and EXPDIR paths)::

      ./setup_expt.py forecast-only --pslot test --idate 2020010100 --edate 2020010118 --resdet 384 --gfs_cyc 4 --comrot /work/noaa/marine/Cameron.Book/ufs/COMROT --expdir /work/noaa/marine/Cameron.Book/ufs/EXPDIR

6. Copy IC files into COMROT/$PSLOT. Directory name should be like::
     
      gfs.20220101, with structure: gfs.$YYYYMMDD/CC/atmos. INPUT folder within …/atmos/ contains sfc files needed for GFS ATM to run.

7. Edit config.base in $EXPDIR/$PSLOT (ACCOUNT, HOMEDIR, STMP/PTMP, HPSSARCH)

8. Run ./setup_workflow_fcstonly.py --expdir $EXPDIR/$PSLOT

   This will generate crontab and xml files for the experiment in $EXPDIR/$PSLOT.

9.  Submit job through crontab by copying entry in $PSLOT.crontab into crontab via crontab -e.

10. Monitor status of workflow using rocotostat::
      
      rocotostat -d /path/to/workflow/database/file -w /path/to/workflow/xml/file [-c YYYYMMDDCCmm,[YYYYMMDDCCmm,...]] [-t taskname,[taskname,...]] [-s] [-T]
      e.g.: rocotostat -d $PSLOT.db -w $PSLOT.xml

11. Check status of specific task/job::
      
      rocotocheck -d /path/to/workflow/database/file -w /path/to/workflow/xml/file -c YYYYMMDDCCmm -t taskname

    Helpful documentation:
    https://github.com/NOAA-EMC/global-workflow/wiki/Run-Global-Workflow





