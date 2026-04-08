# WaveWatch III – Installation & Compilation

## 1) Downloading WaveWatch III source code
Purpose: Download a specific official version of WaveWatch III from GitHub.

curl -L -O https://github.com/NOAA-EMC/WW3/archive/refs/tags/6.07.1.zip
unzip 6.07.1.zip

This downloads and extracts WW3 v6.07.1.

## 2) Checking shared libraries used by executables
Purpose: Verify which system libraries an executable is linked against (useful for debugging runtime errors).

ldd ww3_shel

## 3) Compiling WaveWatch III

a) Standard compilation workflow
./w3_setup ~/WW3-6.07.1/model

b) Set NetCDF environment:

export WWATCH3_NETCDF=NC4
export NETCDF_CONFIG=/apps/NETCDF-FORTRAN/4.5.3/bin/nf-config

c) Select machine and switch:

./w3_setup ~/WW3-6.07.1/model/ -c ionio -s Ifremer1

Mandatory machine-specific files:
comp.ionio, comp.link

## 4) Compile executables:

a) Manual compilation

./w3_make ww3_grid ww3_prnc ww3_strt ww3_shel ww3_ounf ww3_ounp

b) Automake compilation

./w3_automake

All executables will appear in model/exe, together with copies of the switch, comp, and link files used.
