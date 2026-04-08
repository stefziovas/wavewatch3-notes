# Running WaveWatch III on the ARIS Supercomputer

1) Login & Logout
Login: ssh aris
Logout: exit

2) Transferring files to and from ARIS

⚠️ Important
These commands should be executed from the Ionio server, not from ARIS itself,
since outbound connections may be restricted on compute nodes.

a) Send files from Ionio → ARIS
scp ERA5_forcing.nc aris:~/stefz/for-ww3-compile-ARIS/run_with_currents

b) Retrieve files from ARIS → Ionio
scp aris:~/stefz/for-ww3-compile-ARIS/run_with_currents/ERA5_forcing.nc .

3) Checking project and account status

Check available computational budget
mybudget

Useful to verify remaining CPU hours before submitting large jobs.

4) Submitting jobs on ARIS

Submit a batch job
sbatch run_model.sh

This submits the job to the SLURM scheduler.

5) Monitoring running jobs

a) Check jobs for a specific user
squeue -u sofianos

b) Check all jobs on the system
squeue

6) SLURM job submission basics

a) Submitting a job
sbatch my_script.sh

b) Batch script components

A SLURM batch script typically contains: 

i) Scheduler directives: Lines beginning with #SBATCH

ii) Shell commands: Standard UNIX (bash) commands

iii) Job steps: Created using srun

SLURM automatically provides environment variables such as: SLURM_JOBID, SLURM_NODELIST, SLURM_NTASKS, SLURM_CPUS_PER_TASK

7) Minimal SLURM script example

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

8) Job types supported on ARIS

Common runtime models include:

a) Serial jobs – single-core programs

b) MPI jobs – multi-process parallel programs

c) Hybrid jobs – MPI + OpenMP

d) GPU jobs – GPU-accelerated workloads

e) PHI jobs – Intel PHI (offload mode)

f) Multiple serial jobs – several serial programs in one script

WaveWatch III typically runs as an MPI job.

9) SLURM script for running WaveWatch III on ARIS

Example production script

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

10) ## Performance & Scalability Tests

Why this matters: Before launching long WaveWatch III simulations on ARIS supercomputer, it is essential to benchmark computational performance.

This helps: a) optimize processor allocation, b) estimate total wall-clock time, c) avoid wasting project budget, and d)choose output frequency wisely

The tests below were performed prior to the final thesis simulations.

Test configuration (common to all runs)

1) Model: WaveWatch III
2) Domain: Mediterranean Sea
3) Spatial resolution: 1/36° (~ 3km)
4) Time step: 360 s (global)
5) Simulation year: 2020
6) Output frequency: 30 min – 3 h

Setup: very high computational demand

a) Scalability with number of processors

The following tests explore how model progress scales with increasing MPI task counts for a fixed wall-clock runtime of 2 hours.

Model progress achieved in 2 hours of wall time

| MPI tasks | Job ID         | Output frequency | Simulated time |
| --------: | -------------- | ---------------- | -------------- |
|        80 | no_cur.1812994 | 30 min           | ~30 hours      |
|       120 | no_cur.1813150 | 1 hour           | ~40 hours      |
|       160 | no_cur.1813211 | 1 hour           | ~49 hours      |
|       160 | no_cur.1813325 | 3 hours          | ~49.5 hours    |
|       200 | no_cur.1813425 | 1 hour           | ~56 hours      |

Key takeaway
Scaling is positive up to ~160–200 MPI tasks, but efficiency gains begin to flatten, especially when I/O (model's input/output) load increases.

b) Configuration computational cost estimates

Here two different configurations were tested: with and without the inclusion of currents forcing. The tests were made for the duration of one month, in order to take a fast and reliable output for the impact of the two different experiments on the computational time. This is a common approach for planning annual or multi-year runs.

Configuration	MPI tasks	Output frequency	Wall-clock time
no currents	200	        1 hour			~15 hours
with currents	200	        1 hour			~17 hours

⚠️ Initial estimates (~10 h) were corrected after full diagnostic runs.

Practical conclusions

1) 200 MPI tasks offers a good balance between speed and resource usage
2) Output frequency has a non-negligible I/O cost
3) Including currents slightly increases computational demand
4) Annual Mediterranean simulations at 1/36° resolution require careful scheduling

