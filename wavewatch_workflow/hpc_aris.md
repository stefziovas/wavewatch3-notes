# Running WaveWatch III on the ARIS Supercomputer

## Login & Logout
- Login: ```ssh aris```
- Login with ncview enabled: ```ssh -X aris```
- Logout: ```exit```

## Transferring files to and from ARIS

⚠️ Important
These commands should be executed from your own server, not from ARIS itself, since outbound connections may be restricted on compute nodes.

- Send files from your local computer → ARIS
```
scp ERA5_forcing.nc aris:~/stefz/run_with_currents
```
- Retrieve files from ARIS → local computer
```
scp aris:~/stefz/run_with_currents/ERA5_forcing.nc .
```
## Checking project and account status

Check available computational budget
```
mybudget
```
Useful to verify remaining CPU hours before submitting large jobs.

## Submitting jobs on ARIS
```
sbatch run_model.sh
```
This submits the job to the SLURM scheduler.

## Monitoring running jobs

Check jobs for a specific user
```
squeue -u <username>
```

## SLURM job submission basics

- Submitting a job
```
sbatch my_script.sh
```
- Batch script components

A SLURM batch script typically contains: 

1) Scheduler directives: Lines beginning with #SBATCH

2) Shell commands: Standard UNIX (bash) commands

3) Job steps: Created using srun

SLURM automatically provides environment variables such as: SLURM_JOBID, SLURM_NODELIST, SLURM_NTASKS, SLURM_CPUS_PER_TASK

## Minimal SLURM script example
```
#!/bin/bash -l

#SBATCH --job-name=slurm_env
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=12
#SBATCH --mem-per-cpu=1024
#SBATCH --time=00:01:00
#SBATCH --error=job.%J.out
#SBATCH --output=job.%J.out

echo "Start at $(date)"
echo "Running on hosts: $SLURM_NODELIST"
echo "Running on $SLURM_NNODES nodes"
echo "Tasks per node: $SLURM_NTASKS_PER_NODE"
echo "Job ID: $SLURM_JOBID"
echo "End at $(date)"
```
## Job types supported on ARIS

Common runtime models include:

- Serial jobs – single-core programs

- MPI jobs – multi-process parallel programs

- Hybrid jobs – MPI + OpenMP

- GPU jobs – GPU-accelerated workloads

- PHI jobs – Intel PHI (offload mode)

- Multiple serial jobs – several serial programs in one script

WaveWatch III typically runs as an MPI job.

## SLURM script for running WaveWatch III on ARIS
```
#!/bin/bash -l

####################################
#     ARIS SLURM script template   #
####################################

#SBATCH --job-name=no_cur
#SBATCH --output=no_cur.%j.out
#SBATCH --error=no_cur.%j.err
#SBATCH --ntasks=200
#SBATCH --nodes=10
#SBATCH --ntasks-per-node=20
#SBATCH --cpus-per-task=1
#SBATCH --time=02:00:00
#SBATCH --mem=56G
#SBATCH --partition=compute
#SBATCH --account=pr017024_thin

# MPI / stack settings
export I_MPI_FABRICS=shm:dapl
ulimit -s unlimited

# OpenMP configuration
if [ x$SLURM_CPUS_PER_TASK == x ]; then
  export OMP_NUM_THREADS=1
else
  export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
fi

# Load modules
module purge
module load intel/18.0.1 \
            intelmpi/2018.1 \
            gnu \
            netcdf/4.4.1/intel \
            hdf5/1.8.17/intel \
            szip/2.1 \
            jasper

export LD_LIBRARY_PATH=$NETCDF/lib:$HDF5/lib:$JASPERLIB:$LD_LIBRARY_PATH

# Run WaveWatch III
cd /users/pa006/sofianos/stefz/run_no_currents

echo "Running grid"
time ./ww3_grid

echo "Running initial conditions"
time ./ww3_strt

echo "Preparing wind input"
time ./ww3_prnc

echo "Running main model"
time srun -n 200 ./ww3_shel
```

