# Forcing Data & NetCDF Handling

WaveWatch 3 requires at least wind forcing, while also surface currents are one of the most common used, due to their significant impact on waves evolution.
Here we give the common workflow to download data from ERA5 and CMEMS services, while also some basics commands for their proper inclusion in WW3 as forcings. 

## how to download CMEMS data

first, you gonna need module load python3

a) cmems: 1) pip install copernicusmarine
2) touch login.sh and add via vi login.sh: 
import copernicusmarine

copernicusmarine login
3) bash login.sh --> type your username, code
4) touch download_data.py 
5) vi download_data.sh and type your script
6) bash download_data.sh

## how to download ERA5 data

b) era5: 1) vi ~/.cdsapirc and then enter your (or Sofianos) apikey:
url: https://cds.climate.copernicus.eu/api
key: a6d85648-b7b0-4ffb-b87b-a9d31446bfd9
2) $ pip install "cdsapi>=0.7.7"
3) touch download_data.py 
4) vi download_data.py and type your script
5) python3 download_data.py (your gonna need first to module load python)



## GRIB → NetCDF conversion (recommended)
```
cdo -f nc copy ERA5_forcing ERA5_forcing.nc
```
GRIB is preferred when downloading ERA5; NetCDF export is still experimental.

## Editing calendar and time units
```
ncatted -h -a calendar,time,c,c,'standard' wind.nc
ncatted -h -a units,time,o,c,'hours since 1900-01-01T00:00:00Z' wind.nc
```
Be careful to use the correct reference date.

## Reordering latitude and longitude dimensions
```
ncpdq -h -O -a -lat ERA5_forcing.nc ERA5_forcing.nc
```
Used when variables depend on (time, lat, lon) but the file is (time, lon, lat).

## Handling missing _FillValue

WaveWatch III requires ```_FillValue = -32767s``` even if it does not appear in ncdump -h.

Example using CDO:
```
cdo -setattribute,10u@_FillValue=-32767s \
    -setattribute,10v@_FillValue=-32767s \
    ERA5_forcing.nc ERA5_forcing.nc_
```
Then overwrite:
```
mv ERA5_forcing.nc_ ERA5_forcing.nc
```
## Attribute editor (ncatted)
```
ncatted -O \
  -a _FillValue,10u,a,c,-32767s \
  -a _FillValue,10v,a,c,-32767s \
  file1.nc file2.nc
```
## Change variable name
```
cdo -chname,temp,t2m infile outfile
```
## Count zeros in a specific variable
```
ncap2 -O -s 'zero_count=(my_variable == 0).total()' infile.nc count.nc
ncdump count.nc
```
Reference: https://linux.die.net/man/1/ncatted
