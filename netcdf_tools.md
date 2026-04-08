# NetCDF Tools & Inspection

## Inspect a NetCDF file
```
ncdump -h ERA5_forcing.nc
```
Shows: variable names, units, dimensions, attributes, calendar and conventions. You will need the module nco, which you can load as ```module load nco```

You can also, check netcdf format: ```ncdump -k file.nc```

## Quick visualization of NetCDF files
```
ncview <file_name>.nc
```
Requires X-forwarding enabled. For example, if you want to use ncview in a remote connection (e.g. in your labs units or in a hpc system), you should connect using the -X key

```
ssh -X aris # for a connection with enabled ncview option in ARIS hpc system 
```

## File information and statistics
```
cdo sinfo <file_name>
```
You will need the module cdo, which you can load as ```module load cdo```
## Merge NetCDF files along time
- Merging according to dates
```
cdo mergetime input_1.nc input_2.nc output.nc
```
cdo mergetime joins files with the same variables over consecutive, non-overlapping time periods into one long time series. If you have files with non-consecutive dates (e.g., file_1990.nc and file_1995.nc), mergetime will combine them and create a time series with a gap between them. 
```
cdo mergetime file_1990.nc file_1995.nc output_merged.nc
```
Although, if your files have overlapping time steps (e.g., file 1 ends in Dec 1995 and file 2 starts in Dec 1995), cdo mergetime may error out or produce unexpected results.

- Merging files with the same time step
```
cdo merge input_1.nc input_2.nc output.nc
```
Used to merge files that have the same time steps but different variables (e.g., merging a Temperature file and a Precipitation file for the same time period).

## Compatibility issues of the netcdf4 and hdf5 libraries
 ```
nccopy -k classic with_correction/work/ww3_with_wnd_cor202009.nc tmp1.nc
```
Forces classic format in netcdf. With classic format usually you can overpass some issues with the compatibility of the netcdf4 and hdf5 libraries.
