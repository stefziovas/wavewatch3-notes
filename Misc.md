# Miscellaneous Useful Commands

## Search inside files

- Searching for key word: OUTS in all files
```
grep -r "OUTS" *
```

- Searching for key word: OUTS only in switch files
```
grep -r "OUTS" switch_*
```

- If you aren't sure if it's "OUTS", "outs", or "Outs", add the -i flag:
```
grep -i "OUTS" switch_*
```

- If you only need a list of the files that contain the word, add the -l flag:
```
grep -l "OUTS" switch_*
```

## Check directory/files size, disk room and RAM usage 

```
du -sh work/*
```
The flag -s, you avoid printing the size of each file inside searched directory. If you want to check the size of all files inside the directory, you can simpy type ```du -h work/*```.

```
df -h
```
Check available disk size.

```
free -h
```
Shows total, used, and available RAM.

## Linking files (symbolic links)
Purpose: Avoid copying large files and keep consistent filenames across runs.
```
ln -sf <source_file> <target>
```
Examples:

- ```ln -sf outputs/jan/restart001.ww3 restart.ww3```

Links restart001.ww3 to restart.ww3 in the current directory.

- ```ln -sf ../../mod_def.ww3 .```

Links mod_def.ww3 from a parent directory into the current one.

## Search command history

```
history | grep curl
```

## Running WaveWatch III with MPI
```
mpirun -n 4 ./ww3_shel
```
Purpose: Run WW3 using multiple processors.

## Running jobs offline (background execution)
```nohup``` disconnects a process from the terminal on which is run, enabling you to close the connection while the calculation continues to go on.
```
nohup mpirun -n 4 ./ww3_shel &> log.ww3 &
```
Purpose: Run simulations without keeping the terminal open and redirect output to a log file.

## Display the first few or last rows of a file 

- ```head -n 10 FILE```
- ```tail -n 10 FILE```

## Monitoring WW3 outputs
```
tail log.ww3
```

## Stopping a running job

- Check running processes
```
ps aux | grep <username>
```
- Kill a specific process
```
kill -9 <PID>
```
## Interrupt a running program
```
Ctrl + C
```

## Stop a bash program if any command fails

You should use in the start of any bash program ```#!/bin/bash -e```

## Fix segmentation fault due to stack size
Purpose: Avoid forrtl: severe (174): SIGSEGV errors.
```
ulimit -Ss unlimited
```
1) ulimit -s unlimited → sets both soft and hard stack limits to unlimited (if you have permission).

2) ulimit -S -s unlimited → sets only the soft stack limit to unlimited (within the existing hard limit).

On hpc systems, like that of ARIS, ulimit -s unlimited try to adjust the hard limit, which the system may forbids, and you ended up with an inadequate stack size. Using ulimit -S -s unlimited is safer and usually the recommended way on shared supercomputers.
