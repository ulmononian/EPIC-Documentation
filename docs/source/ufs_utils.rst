Using UFS_UTILS to prepare Initial Conditions for MRW
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Cold Start (free-forecast only)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Utilize UFS_UTILS to retrieve GFSv16 (FV3GFS) files from HPSS and convert them to lower resolution / different number of levels than operational GFS (C768C384L127). If the user wishes to simply retrieve C768 files, no conversion is needed, they can be retrieved from HPSS directly or the chgres option in the UFS_UTILS gdas_init config can be turned off. 
To run UFS_UTILS (also available in build of ufs-mrw within the global-workflow checkout)::
    git clone --recursive https://github.com/NOAA-EMC/UFS_UTILS.git
    sh build_all.sh
    cd fix
    sh link_fixdirs.sh emc $MACHINE
    cd ../util/gdas_init
where $MACHINE is Orion, Hera, etc.. 
Within …/utils/gdas_init, adjust config file to set desired resolution/levels of files to be converted, output data directory, etc.. $CRES_HIRES (desired resolution, if different than C768), $LEVS (number of hybrid layers), $RUN_CHGRES (To run chgres, set to 'yes'. To extract data only, set to 'no'.), $EXTRACT_DATA (Set to 'yes' to extract data from HPSS. If data has been extracted and is located in $EXTRACT_DIR, set to 'no'.). More details can be found in header of config. 
Run the driver to submit the HPSS retrieval job and chgres job via::
    ./driver.$MACHINE.sh
After jobs run, converted files should be located in the output directory specified in config. If not, check log files in …/util/gdas_init. An example of files converted from C768 to C384 and C96 can be found here: /scratch2/NCEPDEV/marine/Cameron.Book/gfs_data/chgres. The corresponding UFS_UTILS configuration used to do the retrievals/conversions (where the config settings can also be viewed) is here: /scratch2/NCEPDEV/marine/Cameron.Book/UFS_UTILS/util/gdas_init.
