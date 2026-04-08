# NetCDF Tools & Inspection

## 1) Inspect NetCDF metadata

ncdump -h ERA5_forcing.nc

Shows: variable names, units, dimensions, attributes, calendar and conventions

## 2) Quick visualization of NetCDF files

ncview <file_name>.nc

Requires X-forwarding enabled.  

## 3) File information and statistics

cdo sinfo <file_name>

## 4) Merge NetCDF files along time

cdo mergetime input_1.nc input_2.nc output.nc
