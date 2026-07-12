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
