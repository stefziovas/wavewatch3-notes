# Miscellaneous Useful Commands

## Search command history

```
history | grep curl
```

## Search inside files

- searching for key word: OUTS in all files
```
grep -r OUTS *
```

- for only searching in switch files
```
grep "OUTS" switch_*
```

- If you aren't sure if it's "OUTS", "outs", or "Outs", add the -i flag:
```
grep -i "OUTS" switch_*
```

- If you only need a list of the files that contain the word, add the -l flag:
```
grep -l "OUTS" switch_*
```

## Check file size

```
du -sh wind.ww3
```

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

## Running WaveWatch III with MPI
Purpose: Run the model using multiple processors.
```
mpirun -n 4 ./ww3_shel
```
## Running jobs offline (background execution)
Purpose: Run simulations without keeping the terminal open and redirect output to a log file.
```
nohup mpirun -n 4 ./ww3_shel &> log.ww3 &
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
## Fix segmentation fault due to stack size
Purpose: Avoid forrtl: severe (174): SIGSEGV errors.
```
ulimit -Ss unlimited
```
notes: 1) ulimit -s unlimited → sets both soft and hard stack limits to unlimited (if you have permission).
2) ulimit -S -s unlimited → sets only the soft stack limit to unlimited (within the existing hard limit).

ulimit -s unlimited on ARIS try to adjust the hard limit, which the system may forbids, and you ended up with an inadequate stack size. Using ulimit -S -s unlimited is safer and usually the recommended way on shared supercomputers.
