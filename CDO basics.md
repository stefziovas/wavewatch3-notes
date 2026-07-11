# CDO-Based Analysis for WaveWatch III Outputs

## File information and statistics

```
cdo showname input.nc
```
Shows the name of all variables inside the netcdf. 
```
cdo sinfo input.nc
```
Shows useful information and dates covered by netcdf.
```
cdo sinfon input.nc
```
Shows information and stats (max, min and mean values) for each variable in netcdf.

## Select variables and rename them
```
cdo select,name=hs input.nc output.nc
```
The output file gonna include data for only the SWH.

```
cdo chname,hs,hs_new input.nc output.nc
```

The variable hs is renamed as hs_new and saved in output.nc file.

```
cdo -chname,hs,hs_new -select,name=hs input.nc output.nc
```
Here we combined the last two commmands into a single one, by using the flag "-", where the command neareast the input.nc file is gonna be run first.

## Select specific timesteps or dates
```
cdo seltimestep,1/24 input.nc output.nc
```
Selects the first 24 time steps from the input.nc and save them in output.nc

```
cdo seldate,2020-09-16T00:00:00,2020-09-19T00:00:00 input.nc output.nc 
```
Select data by dates.

## Select a specific region
```
cdo sellonlatbox,lon_1,lon_2,lat_1,lat_2 input.nc output.nc
```
Select data withing a region by specifing lon-lat coordinates.

```
cdo selindexbox,xmin,xmax,ymin,ymax infile.nc outfile.nc
```

Select data withing a region by specifing the indecies of grid in the domain. Note: You can translate indicies into lat-lon and vise-versa by using a little python script via kd-tree.

## Add a file or a variable into a file
```
ncks -A file_1.nc file_2.nc
```
Add a file_1.nc into a file_2.nc
```
ncks -v hs file_1.nc file_2.nc
```
Add variable hs from file_1.nc to file_2.nc

## Compute mean and std values
```
cdo timavg input.nc output_mean.nc
cdo timstd input.nc output_std.nc
```
Average and std values for all the time steps inside input.nc file.

```
cdo monmean input.nc output_mon_mean.nc
cdo monstd input.nc output_mon_std.nc
```
Monthly average and std values of input.nc file, saved in output.nc.
```
cdo seasmean input.nc output_seas_mean.nc
cdo seasstd input.nc output_seas_std.nc
```
Seasonal average and std values of input.nc file, saved in output.nc.

## Compute differences between two files
```
cdo sub input_1.nc input_2.nc diff.nc
```

## Multiple with files and constants
```
cdo mul file_1.nc file_2.nc output.nc
cdo mulc,10 input.nc output.nc
cdo div file_1.nc file_2.nc output.nc
cdo divc,10 input.nc output.nc
```

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

## Computing derived variables using exprf

During exprf, cdo reads a txt file with the instructions about the desired calculations

- Computing wind speed and direction (context of exprf file: wind_exprf.txt)
```
speed = sqrt(uwnd^2 + vwnd^2);
dir   = atan(vwnd / uwnd);
```
Apply expressions
```
cdo exprf,wind_exprf.txt ww3.wind.nc ww3.windchar.nc
ncks -A ww3.windchar.nc ww3.wind.nc
``` 

- Compute wind–current relative angle (context of exprf file: wnd_cur_exprf.txt)
```
wnd_speed = sqrt(uwnd^2 + vwnd^2);
cur_speed = sqrt(ucur^2 + vcur^2);

dir_wnd = vwnd>0&&uwnd>=0?atan(uwnd/vwnd):
          (vwnd>0&&uwnd<0?atan(uwnd/vwnd)+6.28:
          (vwnd<0?atan(uwnd/vwnd)+3.14:
          (vwnd==0&&uwnd>=0?3.14/2:-3.14/2)));

dir_wnd_deg = deg(dir_wnd);

dir_cur = vcur>0&&ucur>=0?atan(ucur/vcur):
          (vcur>0&&ucur<0?atan(ucur/vcur)+6.28:
          (vcur<0?atan(ucur/vcur)+3.14:
          (vcur==0&&ucur>=0?3.14/2:-3.14/2)));

dir_cur_deg = deg(dir_cur);

angle_bet_wnd_cur = abs(dir_wnd_deg - dir_cur_deg);
```

- Apply expressions
```
cdo exprf,wnd_cur_exprf.txt ww3.run_with_cur.nc ww3.wnd_cur_char.nc
```
