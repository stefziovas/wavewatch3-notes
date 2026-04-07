# HPC & Process Management Commands

## 1)  Creating directories
Purpose: Create directory trees safely (no error if they already exist).

mkdir -p outputs/jan

## 2) Linking files (symbolic links)
Purpose: Avoid copying large files and keep consistent filenames across runs.

ln -sf <source_file> <target>

Examples
a) ln -sf outputs/jan/restart001.ww3 restart.ww3

Links restart001.ww3 to restart.ww3 in the current directory.

b) ln -sf ../../mod_def.ww3 .

Links mod_def.ww3 from a parent directory into the current one.

## 3) Running WaveWatch III with MPI
Purpose: Run the model using multiple processors.

mpirun -n 4 ./ww3_shel

## 4) Running jobs offline (background execution)
Purpose: Run simulations without keeping the terminal open and redirect output to a log file.

nohup mpirun -n 4 ./ww3_shel &> log.ww3 &

## 5) Stopping a running job

a) Check running processes

ps aux | grep <username>

b) Kill a specific process

kill -9 <PID>

## 6) Interrupt a running program
Purpose: Stop the run 

Ctrl + C

## 7) Fix segmentation fault due to stack size
Purpose: Avoid forrtl: severe (174): SIGSEGV errors.

ulimit -Ss unlimited

note: 1) ulimit -s unlimited → sets both soft and hard stack limits to unlimited (if you have permission).
2) ulimit -S -s unlimited → sets only the soft stack limit to unlimited (within the existing hard limit).

ulimit -s unlimited on ARIS try to adjust the hard limit, which the system may forbids, and you ended up with an inadequate stack size. Using ulimit -S -s unlimited is safer and usually the recommended way on shared supercomputers.
