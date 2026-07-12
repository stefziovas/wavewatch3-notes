# Running WaveWatch III on ATOS HPC system (ECMWF)

## Login & Logout
- Login: ```ssh hpc-login```
- Login with ncview enabled: ```ssh -X hpc-login```
- Logout: ```exit```

## Transferring files to and from ECMWF

⚠️ Important
These commands should be executed from your own server, not from ECMWF itself, since outbound connections may be restricted on compute nodes.

- Send files from your local computer → ECMWF
```
scp ERA5_forcing.nc hpc-login:~/stefz/run_with_currents
```
- Retrieve files from ECMWF → local computer
```
scp hpc-login:~/stefz/run_with_currents/ERA5_forcing.nc .
```

## Loading modules
- Load modules
```
module load cdo nco ncview python3 
```
- List current loaded modules
```
module list
```
- Search available modules and their versions  
```
module avail cdo 
```
```
module spider cdo
```

## Submitting jobs on ECMWF
```
sbatch run_model.sh
```
This submits the job to the SLURM scheduler.

## Monitoring running jobs

Check jobs for a specific user
```
squeue -u <username>
```

## Cancelling a running job
```
scancel <job-id>
```

## SLURM job submission basics

- Batch script components

A SLURM batch script typically contains: 

1) Scheduler directives: Lines beginning with #SBATCH

2) Shell commands: Standard UNIX (bash) commands

3) Job steps: Created using srun

SLURM automatically provides environment variables such as: SLURM_JOBID, SLURM_NODELIST, SLURM_NTASKS, SLURM_CPUS_PER_TASK

## SLURM scripts for running WaveWatch III on ATOS

# WW3-NEMO coupled configuration 
```
#!/bin/bash

#SBATCH --job-name=MED36_TOP_CPL_y1994      # Job name
#SBATCH --output=./output/logs/%x.%j.out    # Save executables outputs
#SBATCH --error=./output/logs/%x.%j.err     # Save logs errors
#SBATCH --time=15:00:00                     # Walltime
#SBATCH --qos=np
#SBATCH --account=spgrver2                  # Replace with your system project
#SBATCH --mem=230G                          # Memory per NODE
#SBATCH --hint=nomultithread                # Make sure only physical cores are used for performance (disabling multithreading)
#SBATCH --mail-type=ALL
#SBATCH --mail-user=stziovas@phys.uoa.gr    # Send email notifications 
#SBATCH --nodes=16                          # Number of nodes requested (ww3:10, nemo: 5, xios: 1)

set -e  # Exit immediately if a command exits with a non-zero status.

## LOAD MODULES ##
module purge
module load prgenv/intel intel/2021.4.0 hpcx-openmpi/2.9.0 netcdf4-parallel/4.9.3 hdf5-parallel/1.14.6

ulimit -s unlimited

export LD_LIBRARY_PATH=$PERM/oasis3-mct/atos_oa3-mct/lib:$NETCDF4_DIR/lib:$HDF5_DIR/lib:$JASPERLIB:$LD_LIBRARY_PATH

INDIR=$(pwd)
cd $INDIR

# WW3 allocation
BIN_WW3=./ww3_shel
WW3_MPI=320
WW3_OMP=4
WW3_DIST=32
WW3_NODES=10

# NEMO allocation
BIN_NEMO=./nemo
NEMO_MPI=640
NEMO_OMP=1
NEMO_DIST=128
NEMO_NODES=5

# XIOS allocation
BIN_XIOS=./xios_server.exe
XIOS_MPI=64
XIOS_OMP=1
XIOS_DIST=64
XIOS_NODES=1

# OMP exports for ww3 (hybrid MPI-OPNEMP)
export OMP_PLACES=cores
export OMP_PROC_BIND=close

# Run MED36_TOP_CPL
time srun --kill-on-bad-exit=1 \
        --ntasks=${WW3_MPI} --nodes=${WW3_NODES} --cpus-per-task=${WW3_OMP} --cpu-bind=cores --ntasks-per-node=${WW3_DIST} --export=ALL,OMP_NUM_THREADS=${WW3_OMP} ${BIN_WW3} : \
        --ntasks=${NEMO_MPI} --nodes=${NEMO_NODES} --cpus-per-task=${NEMO_OMP} --ntasks-per-node=${NEMO_DIST} ${BIN_NEMO} : \
        --ntasks=${XIOS_MPI} --nodes=${XIOS_NODES} --cpus-per-task=${XIOS_OMP} --ntasks-per-node=${XIOS_DIST}  ${BIN_XIOS}

## ACCOUNTING ##
sacct --format=jobid,jobname,partition,ntasks,elapsed,MaxRSS,state -j $SLURM_JOBID
echo "MED36_TOP_CPL run was completed !!"
```

# Running WaveWatch via depedency runs

- Depedency script 
```
#!/bin/bash

Indir=$(pwd)
cd $Indir

PRE=$(sbatch pre_run_ww3.sh | awk '{print $4}')
echo "Preprocessing job: $PRE"

MODEL=$(sbatch --dependency=afterok:$PRE run_ww3.sbatch | awk '{print $4}')
echo "Model job: $MODEL"

POST=$(sbatch --dependency=afterok:$MODEL post_run_ww3.sh | awk '{print $4}')
echo "Postprocessing job: $POST"

echo "Run completed"
```
- Pre run script
```
#!/bin/bash

#SBATCH --job-name=pre_run   # Job name
#SBATCH --output=%x.%j.out   # Save executables outputs
#SBATCH --error=%x.%j.out    # Save logs errors
#SBATCH --ntasks=1           # Number of nodes requested
#SBATCH --time=00:15:00      # Walltime
#SBATCH --mem=128G           # Memory per NODE
#SBATCH --qos=nf
#SBATCH --account=spgrver2   # Replace with your system project

## LOAD MODULES ##
module purge
module load prgenv/intel intel/2021.4.0 hpcx-openmpi/2.9.0 netcdf4-parallel/4.9.3 hdf5-parallel/1.14.6

WORKDIR=$(pwd)
cd $WORKDIR

echo "Setting the grid: ww3_grid"
./ww3_grid > grid_outputs.txt
echo "Making the initial restart file: ww3_strt"
./ww3_strt
# Wind
echo "Preparing wind forcing: ww3_prnc"
ln -sf ww3_prnc.nml_wnd ww3_prnc.nml
./ww3_prnc
# Currents
echo "Preparing surface currents forcing: ww3_prnc"
ln -sf ww3_prnc.nml_cur ww3_prnc.nml
./ww3_prnc
```

- Main script

```
#!/bin/bash

#SBATCH --job-name=ww3_run     # Job name
#SBATCH --output=%x.%j.out     # Save executables outputs
#SBATCH --error=%x.%j.out      # Save logs errors
#SBATCH --nodes=10             # Number of nodes requested
#SBATCH --ntasks=320           # Total tasks (Total MPI cores)
#SBATCH --ntasks-per-node=32   # Tasks per node (MPI cores per node)
#SBATCH --cpus-per-task=4      # OPENMP threads
#SBATCH --time=00:20:00        # Walltime
#SBATCH --qos=np 
#SBATCH --mem=230G             # Memory per NODE
#SBATCH --hint=nomultithread   # Make sure only physical cores are used for performance (disabling multithreading)
#SBATCH --account=spgrver2     # Replace with your system project

module purge
module load prgenv/intel intel/2021.4.0 hpcx-openmpi/2.9.0 netcdf4-parallel/4.9.3 hdf5-parallel/1.14.6

ulimit -s unlimited

export LD_LIBRARY_PATH=$NETCDF4_DIR/lib:$HDF5_DIR/lib:$JASPERLIB:$LD_LIBRARY_PATH

# OpenMP
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export OMP_PLACES=cores
export OMP_PROC_BIND=close

WORKDIR=$(pwd)
cd $WORKDIR

srun --cpu-bind=cores ./ww3_shel
```
- Post process script

```
#!/bin/bash

#SBATCH --job-name=post_run_test_fields   # Job name
#SBATCH --output=%x.%j.out                # Save executables outputs
#SBATCH --error=%x.%j.out                 # Save logs errors
#SBATCH --ntasks=1                        # Number of nodes requested
#SBATCH --time=00:45:00                   # Walltime
#SBATCH --mem=128G                        # Memory per NODE
#SBATCH --qos=nf
#SBATCH --account=spgrver2                # Replace with your system project

## LOAD MODULES ##
module purge
module load prgenv/intel intel/2021.4.0 hpcx-openmpi/2.9.0 netcdf4-parallel/4.9.3 hdf5-parallel/1.14.6

WORKDIR=$(pwd)
cd $WORKDIR

echo "Running Post-process: ww3_ounf"
./ww3_ounf
echo "Run completed"
```
