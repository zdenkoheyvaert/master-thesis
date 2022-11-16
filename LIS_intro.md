Getting started with LIS
===============

I stored all necessary files in `/staging/leuven/stg_00024/MSC_TMP/amandine/lis_intro`. Copy them to a new folder in your data directory.

## Overview of the input data
Let's have a look at all the files that are in the directory.

* The `lis.config` file contains all the settings of the run (references to input data, specifying the domain and period, ...). We will discuss it in the next section.

* In the `./ldt` folder you will find several files. The LDT executable is used to prepare a file that can serve as input to LIS. I already did this for you, the resulting file is called `lis_input.nc`. This is the only one you will really need. Have a look at it through
```
module load ncview
ncview ./ldt/lis_input.nc
```

* In the `./_INPUT_files_` folder we have the directory linking to the meteorological forcing data.

* In the `./_INPUT_param_` folder we have two files: `forcing_variables.txt` specifies which meteorological variables are used to force the land surface model. The file `MODEL_OUPUT_LIST.TBL` specifies which variables should be in the output. Have a look at both of them.
You can decrease the number of variables in `MODEL_OUPUT_LIST.TBL` to speed up the simulation and/or not to fill up too much space.

## The `lis.config` file
Don't forget to change the following setting:
```
Output directory: /scratch/leuven/340/vsc34049/lis_intro
```
This is the directory where LIS should store the output. Change it to a folder on your own scratch.

Other settings you may want to change:

* The output interval. Typically `"1da"` for a normal simulation and `"1mo"` for a spinup run:
```
Surface model output interval: "1da"
```

* The start mode. Use `"coldstart"` for a spinup run and `"restart"` to resume a previous simulation:
```
Start mode:	"coldstart"
```

* The start and end of the simulation:
```
Starting year:                                   1990
Starting month:                                  1
Starting day:                                    1
Starting hour:                                   0
Starting minute:                                 0
Starting second:                                 0
Ending year:                                     2002
Ending month:                                    1
Ending day:                                      1
Ending hour:                                     0
Ending minute:                                   0
Ending second:                                   0
```

* The restart file: `"none"` for a cold start but if you want to resume a simulation (e.g., after an initial spinup run) you should link to the correct file here:
```
Noah-MP.4.0.1 restart file: "none"
```

## Running LIS
### On an interactive node
Before we can run LIS, we should load in all necessary libraries. You can do this by running
```
source /data/leuven/340/vsc34049/modules/KUL_LIS_modules
```
As you will have to do this often, it's a good idea to make an alias for this command.

On an interactive node, you can run LIS via
```
./LIS -f lis.config
```
Or, if you want to run LIS in parallel mode (using multiple processors):
```
ulimit -s unlimited
mpirun -np 6 ./LIS -f lis.config 
```

Here `./LIS` is the executable and `lis.config` is the config file.

### Through submitting a job
For long runs, you don't want to keep the terminal open with an interactive session the whole time. In this case you should submit your LIS simulation as a job to the HPC.

To this end, create a file `run.pbs` in the same folder as your `lis.config` file with the following structure:
```
#! /bin/bash
#PBS -l walltime=01:23:59:00
#PBS -l nodes=1:ppn=36:skylake
#PBS -W x=excludenodes=r23i13n23
#PBS -A lt1_col_2021_1
#PBS -N my_custom_run_name
#PBS -m abe
#PBS -M my.email@address.be
#PBS -o ./log.txt
#PBS -e ./out.txt

source /data/leuven/340/vsc34049/modules/KUL_LIS_modules
cd /scratch/leuven/340/vsc34049/files_for_amandine/lis_intro

ulimit -s unlimited
mpirun -np 36 ./LIS -f lis.config

```

Submit the job on a login node through
```
qsub run.pbs
```

You will receive an e-mail when your job starts and finishes.

Good to know:

* The e-mail when your job is completed contains an 'exit code'. If this code is not 0, an error occurred and the simulation crashed / ended prematurely.

* Each job has a 'job id', which is shown after you run `qsub`. You can run `showstart <jobid>` to get an approximation of when the job will start (depending on how busy the queue is), and `qdel <jobid>` to cancel the run (credits will then be reimbursed to your account).

Visit [the documentation on our group's GitHub page](https://github.com/KUL-RSDA/documentation/blob/master/LIS/LIS_on_HPC.md) for more details on running LIS on the HPC.