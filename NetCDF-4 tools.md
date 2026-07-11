# NetCDF Tools & Inspection

## Inspect a NetCDF file
```
ncdump -h ERA5_forcing.nc
```
Shows: variable names, units, dimensions, attributes, calendar and conventions. 

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
Shows informations and stats (max, min and mean values) for each variable in netcdf.
```
ncdump -v <variable> <file_name>
```
Shows useful informations and the values of the desired variable at each index (e.g. ```ncdump -v longitude MED36_dom.nc``` shows longitude values at each grid point of the domain. 
```
cdo showname <file_name>
```
Shows the name of all variables inside the netcdf.

You will need the modules cdo and nco, which you can install in your system or if you are working in a hpc system, you can load them as ```module load cdo nco```.

## Merge NetCDF files along time
- Merging according to dates
```
cdo mergetime input_1.nc input_2.nc output.nc
```
cdo mergetime joins files with the same variables over consecutive, non-overlapping time periods into one long time series. If you have files with non-consecutive dates (e.g., file_1990.nc and file_1995.nc), mergetime will combine them and create a time series with a gap between them. 
```
cdo mergetime file_1990.nc file_1995.nc output_merged.nc
```
Although, if your files have overlapping time steps (e.g., file 1 ends in Dec 1995 and file 2 starts in Dec 1995), cdo mergetime may produce unexpected results, like duplicate dates in your final file. In these cases, you can use ```export SKIP_SAME_TIME=1``` before ```cdo mergetime``` to avoid duplicate dates.

- Merging files with the same time step
```
cdo merge input_1.nc input_2.nc output.nc
```
Used to merge files that have the same time steps, but different variables (e.g., merging a Temperature file and a Precipitation file for the same time period).

- Attach files into one merged file
```
cdo cat input_1.nc input_2.nc output.nc
```
Used to concatenate datasets sequentially (stack file input_1 into the end of input_2).

## GRIB → NetCDF conversion
- Via cdo
```
cdo -f nc copy ERA5_forcing ERA5_forcing.nc
```
- Via xarray lib in python
```
import xarray as xr

ds = xr.open_dataset("ERA5_forcing", engine="cfgrib")
ds.to_netcdf("ERA5_forcing.nc")
```
## Compatibility issues of the netcdf4 and hdf5 libraries

It is recommented to first test different earlier versions of the tools you are using (e.g. cdo, nco), because latest versions might have unresolved issues with the format of the netcdf you are using. An alternative solution is to chenge the netcdf format to cleasic: 

 ```
nccopy -k classic with_correction/work/ww3_with_wnd_cor202009.nc tmp1.nc
```
Forces classic format in netcdf. With classic format usually you can overpass some issues with the compatibility of the netcdf4 and hdf5 libraries.
