---
layout: post
title: "Running the CESM on the SNOW cluster at UAlbany"
categories:
---

## A brief tutorial for students in ATM 623, Spring 2017

## Accessing the Snow cluster

Hopefully you all have an account on Snow already. If not, email me.

```
ssh -Y [yourNetID]@headnode.rit.albany.edu
```

This gets you onto the *headnode*, which is where you will submit your runs to the cluster.

To see what is currently running:

```
squeue
```

To see some general information about the available queues and nodes on the cluster:

```
sinfo
```

## Setup environment variables

Add the following line to your `.cshrc` file in your home directory:

```
setenv CCSMROOT /network/rit/home/br546577/roselab_rit/cesm/cesm1_2_1
```

The environment variable `$CCSMROOT` now points to CESM source code.

You may want to snoop through Brian's `.cshrc` file at `/network/rit/home/br546577/.cshrc`. Some other settings may be necessary, especially if you encounter compiler problems when building the model.

## Look at the CESM documentation

Here is a link to the [CESM1.2 User's Guide](http://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/usersguide/book1.html)

I will give a brief overview below of how to set up and run a simulation on Snow, but you should read the manual for more details about each step.

## Set up a case with prescribed SST

### Choose a location for your case directory

This should probably be somewhere within your home directory. I will give an example here:

```
cd /network/rit/home/br546577/test_CESM
```

### Create the case

```
/network/rit/lab/roselab_rit/cesm/cesm1_2_1/scripts/create_newcase -mach snow -res f19_g16 -compset F_2000 -case mytest
```

This runs the script called `create_newcase` with the following options:

- `-mach snow` switch sets everything up for the snow cluster
- `-res f19_g16` switch sets the resolution to 2º finite volume for the atmosphere, 1º displaced-pole grid for the ocean and sea ice. This is the resolution I suggest for all your proposed experiments, same as the slab ocean simulations we looked at in class.
- `-compset F_2000` set the model to *Data Ocean* or prescribed SST mode (signified by the `F`), prescribed sea ice, year 2000 SSTs and atmospheric composition, and CAM4 atmospheric physics. There are many more pre-defined compsets available, see [here](http://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/modelnl/compsets.html).
- `-case mytest` just chooses a name for the experiment

### Run the setup script

```
cd mytest
cesm_setup
```

This creates the run script and build script among other things.

### Build the model

First you need to decide where to keep the executable files. If you look in you will find the lines

```
<!--"case executable root directory (executable is $EXEROOT/cesm.exe, component libraries are in $EXEROOT/bld) (char) " -->
<entry id="EXEROOT"   value="/data/rose_scr/$CCSMUSER/cesmruns/$CASE/bld"  />
```

This variable `EXEROOT` determines where the model will be built. By default it will look for a directory within `/data/rose_scr/`. This will only work if you have write access to `/data/rose_scr/`, which is probably only true if you are part of the Rose research group. I suggest you set this to somewhere within your own group's scratch space.

For example, if your NetID is `bb533987` and you are part of the Zhou group, you could do this:

```
xmlchange EXEROOT=/data/zhou_scr/bb533987/cesmruns/$CASE/bld
```

At the same time you should also set the run directory `RUNDIR` (where the output files will be written). Usually this would be another subdirectory alongside the build, e.g:

```
xmlchange RUNDIR=/data/zhou_scr/bb533987/cesmruns/$CASE/run
```

Now you can build the model like this (from your case directory):

```
srun -p snow `pwd`/mytest.build
```

This sends the build script `mytest.build` to the cluster to execute on a compute node. It will probably take 10 minutes or so to finish.

### Get familiar with runtime options

The settings that control how the model runs, for how long, etc, are all found in the file `env_run.xml`. You can see individual settings with the `xmlquery` tool. E.g. how long is the run?

```
xmlquery STOP_OPTION
xmlquery STOP_N
```
will reveal that it is currently (by default) set to run for 5 days (useful for a short test).

You can change any of these settings with the `xmlchange` tool. Here we are going to set it to 1 month:

```
xmlchange STOP_OPTION=nmonths
xmlchange STOP_N=1
```

One thing you should do is switch off the "short-term archiving" option. This will try to move the model output from the run directory to another archive once the run is completed. It will be simpler to disable this:

```
xmlchange DOUT_S=FALSE
```

### The prescribed SST and sea ice

This compset uses prescribed SST and prescribed sea ice. The model reads these from a file at runtime. You can see what file is being used:

```
xmlquery SSTICE_DATA_FILENAME
```

which should produce

```
env_run.xml: SSTICE_DATA_FILENAME = /data/rose_scr/cesm_inputdata/atm/cam/sst/sst_HadOIBl_bc_1x1_clim_c101029.nc
```

It would be a great idea to take a look at this file and see what's in it. Those of you who want to do experiments with different prescribed SST patterns will need to create new files that have your prescribed patterns in them. To set up your perturbation experiments, you would then use `xmlchange` to set `SSTICE_DATA_FILENAME` to point to your custom file.


### Run the model

The default configuration for this resolution is to run on a single node (32 cores), and you will probably not have to change this. To submit your run to the Snow cluster queue, just do this:

```
mytest.submit
```

You can then see if your job is running by doing  

```
squeue
```

### Brief look at the output

The model will write output to the run directory specified in `RUNDIR` (which is also defined in `env_run.xml`). You already set `RUNDIR` during the build step above.

Go the run directory now and see what's there. For me, this would be

```
cd /data/rose_scr/br546577/cesmruns/mytest/run
ls
```

After the 1-month simulation is done (should take about 15 minutes), this produces the following:

```
atm_in                       ice_in                             mytest.clm2.r.0001-02-01-00000.nc     rpointer.atm
atm.log.170316-134219.gz     ice.log.170316-134219.gz           mytest.clm2.rh0.0001-02-01-00000.nc   rpointer.drv
atm_modelio.nml              ice_modelio.nml                    mytest.cpl.r.0001-02-01-00000.nc      rpointer.ice
cesm.log.170316-134219.gz    lnd_in                             mytest.docn.rs1.0001-02-01-00000.bin  rpointer.lnd
cpl.log.170316-134219.gz     lnd.log.170316-134219.gz           mytest.rtm.h0.0001-01.nc              rpointer.ocn
cpl_modelio.nml              lnd_modelio.nml                    mytest.rtm.r.0001-02-01-00000.nc      rpointer.rof
docn_in                      mytest.cam.h0.0001-01.nc           mytest.rtm.rh0.0001-02-01-00000.nc    seq_maps.rc
docn_ocn_in                  mytest.cam.r.0001-02-01-00000.nc   ocn.log.170316-134219.gz              timing
docn.streams.txt.prescribed  mytest.cam.rs.0001-02-01-00000.nc  ocn_modelio.nml                       wav_modelio.nml
drv_flds_in                  mytest.cice.h.0001-01.nc           rof_in
drv_in                       mytest.cice.r.0001-02-01-00000.nc  rof.log.170316-134219.gz
glc_modelio.nml              mytest.clm2.h0.0001-01.nc          rof_modelio.nml
```

The most important things here are the so-called history files:

```
mytest.cam.h0.0001-01.nc
mytest.cice.h.0001-01.nc
mytest.clm2.h0.0001-01.nc
```

respectively the output from the atmospheric model (CAM), the sea ice model (CICE), and the land surface model (CLM).

Each file has a date string in its name, in this case `0001-01` means month 01 of year 0001. By default, CESM produces monthly history files for each component. Of course it is possible to customize how the output is produced. Read the User Manual to find out more.

For now, take a look at these history files and see what's in them. The CAM output at least should look familiar from the homework exercises that you already did.


### Summary: an complete setup and run procedure

Just listing all the above commands in order, assuming you are user `br546577` (which is me). Adapt the username and paths as needed.

```
cd /network/rit/home/br546577/test_CESM
/network/rit/lab/roselab_rit/cesm/cesm1_2_1/scripts/create_newcase -mach snow -res f19_g16 -compset F_2000 -case mytest
cd mytest
cesm_setup
xmlchange EXEROOT=/data/rose_scr/br546577/cesmruns/$CASE/bld
xmlchange RUNDIR=/data/rose_scr/br546577/cesmruns/$CASE/run
srun -p snow `pwd`/mytest.build
xmlchange STOP_OPTION=nmonths
xmlchange STOP_N=1
xmlchange DOUT_S=FALSE
mytest.submit
```

The case root directory is

```
/network/rit/home/br546577/test_CESM/mytest
```

The run directory is

```
/data/rose_scr/br546577/cesmruns/mytest/run
```


## Running the Slab Ocean version

If you want to run the slab ocean rather than prescribed SST, you need one of the `E` compsets (see CESM manual). You also need to specify a file for the slab ocean input (containing information about the prescribed heat source/sink, among other things).

For your reference, here are the commands I originally used to set up the pre-industrial control run that we looked at in class.

*This will run the model for 30 years!* ***Do not repeat this***, *it is for reference only.*

```
/network/rit/lab/roselab_rit/cesm/cesm1_2_1/scripts/create_newcase -mach snow -compset E_1850_CN -res f19_g16 -case som_1850_f19
cd som_1850_f19
cesm_setup
xmlchange DOCN_SOM_FILENAME=pop_frc.gx1v6.100513.nc
srun -p snow -N 1 `pwd`/som_1850_f19.build
xmlchange STOP_OPTION=nyears
xmlchange STOP_N=2
xmlchange RESUBMIT=14
som_1850_f19.submit
```

You will need to adapt this to your own needs. One thing to notice above: we set the run time to 2 years, and use the `RESUBMIT` option to have the model automatically resubmit itself at the end of each 2-year chunk. Setting `RESUBMIT` to 14 means that we get 15 x 2 years = 30 years total simulation.

This is good practice for sharing the cluster. Each 2-year chunk will run in a few hours, and as long as there is a node available, it will continue without interruption. But if the cluster is very busy, the run may wait for a while in the queue before it executes. Using `RESUBMIT` makes this all automatic so you usually don't have to think about it.
