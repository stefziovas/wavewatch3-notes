# WaveWatch III – Installation & Compilation

## Downloading WaveWatch III source code
```
curl -L -O https://github.com/NOAA-EMC/WW3/archive/refs/tags/6.07.1.zip
unzip 6.07.1.zip
```
This downloads and extracts the specific official version v6.07.1 of WaveWatch III from GitHub.

## Compiling WaveWatch III
- Load required modules
```
module purge # clean all current loaded modules 
module load prgenv/intel intel/2021.4.0 intel-mpi/2021.4.0 netcdf4-parallel/4.9.3 hdf5-parallel/1.14.6
```
- Set NetCDF environmental variables
```
export WWATCH3_NETCDF=NC4
export NETCDF_CONFIG=/apps/NETCDF-FORTRAN/4.5.3/bin/nf-config
```
- Initial setup
```
./w3_setup ~/WW3-6.07.1/model
```
- Select machine and switch
```
./w3_setup ~/WW3-6.07.1/model/ -c ATOS -s Ifremer1
```
In order to setup WW3 via your choise of machine architecture (e.g ATOS), you will need to have these mandatory machine-specific files in model/bin directory: comp.ATOS, link.ATOS, where you should include your own compiler options (e.g. ifort, icc). 

Via switch file you can select the physical and numerical schemes that you gonna follow during simulations, but also enable different additional options, such as the ability to run in a coupling mode (e.g. for a NEMO-WW3 configuration: OASIS, OASOCM) using a hybrid MPI-OPENMP set up (OMPG, OMPH).

- Compile executables

1) Manual compilation
```
./w3_make ww3_grid ww3_prnc ww3_strt ww3_shel ww3_ounf ww3_ounp 2>&1 | tee compile.log
```
Redirects Standard Error (stderr) to Standard Output (stdout) and save it into a file named compile.log

2) Automake compilation
```
./w3_automake
```
All executables will appear in model/exe, together with copies of the switch, comp, and link files used.

3) Automake compilation of specific executables
```
./w3_automake ww3_grid ww3_prnc ww3_strt ww3_shel ww3_ounf ww3_ounp
```

**Note**: The only parallel (multi-processors) executables in WaveWatch are the main executables ww3_shel and ww3_multi. All other executables runs serial in one cpu.  
