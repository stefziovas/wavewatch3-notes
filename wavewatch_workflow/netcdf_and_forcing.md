# Forcing Data & NetCDF Handling

WaveWatch III requires at minimum wind forcing, while surface currents are also commonly used due to their significant impact on wave evolution. Here, we outline a typical workflow for downloading data from ERA5 and CMEMS services, along with basic commands for properly incorporating these datasets into WW3 as forcings.
For both sources, Python can be used (see python analysis/forcing_template.ipynb), while CMEMS data can additionally be downloaded via a bash script. 

## How to download CMEMS data

1) Install CMEMS services for download data: ```pip install copernicusmarine```
2) Create a login script to avoid entering your credentials repeatedly: ```touch login.sh``` Edit your login file ```vi login.sh``` and type:
```
import copernicusmarine

copernicusmarine login
```
3) Run the login script and provide your personal credentials (username, password): ```bash login.sh```
4) Create a file to download data: ```touch download_data.sh``` 
5) Edit your file ```vi download_data.sh``` and add your script
```
import copernicusmarine

copernicusmarine.subset(
  dataset_id="cmems_mod_med_phy-cur_my_4.2km_PT1H-m",
  dataset_version="202511",
  variables=["uo", "vo"],
  minimum_longitude=-6,
  maximum_longitude=36.29166793823242,
  minimum_latitude=30.1875,
  maximum_latitude=45.97916793823242,
  start_datetime="2020-09-14T00:00:00",
  end_datetime="2020-09-19T00:00:00",
  coordinates_selection_method="strict-inside",
  netcdf_compression_level=1,
  disable_progress_bar=True,
  output_filename="C://Users//kosta//OneDrive//Desktop//Oceanography//currents_forcing_test_ECMWF_runs.nc"  # where to store the data
)
```
7) Run ```bash download_data.sh``` to download data

## How to download ERA5 data

1) Create a CDS API configuration file:: ```vi ~/.cdsapirc``` and then insert your API key.:
```
url: https://cds.climate.copernicus.eu/api
key: a6d85648-b7b0-4ffb-b87b-a9d31446bfd9
```
3) Install ERA5 API for download data: ```pip install "cdsapi>=0.7.7"```
4) Create a file to download data: ```touch download_data.py``` 
5) Edit your file ```vi download_data.py``` and add your script
```
import cdsapi

dataset = "reanalysis-era5-single-levels"
request = {
    "product_type": ["reanalysis"],
    "variable": [
        "10m_u_component_of_wind",
        "10m_v_component_of_wind"
    ],
    "year": ["2020"],
    "month": [
        "09"
    ],
    "day": [
        "14","15","16", "17", "18", "19"
    ],
    "time": [
        "00:00", "01:00", "02:00",
        "03:00", "04:00", "05:00",
        "06:00", "07:00", "08:00",
        "09:00", "10:00", "11:00",
        "12:00", "13:00", "14:00",
        "15:00", "16:00", "17:00",
        "18:00", "19:00", "20:00",
        "21:00", "22:00", "23:00"
    ],
    "data_format": "grib",
    "download_format": "unarchived",
    "area": [48, -12, 27, 38]
}

client = cdsapi.Client()
client.retrieve(dataset, request).download("C://Users//kosta//OneDrive//Desktop//Oceanography//wind_forcing_test_ECMWF_runs")  # where to store the data
```
6) Run ```python3 download_data.py``` to download data

 **note**: ERA5 data are typically downloaded in GRIB format and then converted to NetCDF. However, downloading directly in NetCDF format is also supported.

## GRIB → NetCDF conversion (recommended)
- Via cdo
```
cdo -f nc copy ERA5_forcing ERA5_forcing.nc
```
- Via python
```
import xarray as xr

ds_1 = xr.open_dataset('C://Users//kosta//OneDrive//Desktop//Oceanography//wind_forcing_test_ECMWF_runs', engine="cfgrib")
ds_1.to_netcdf()
ds_1
```
## Editing calendar and time units
WaveWatch III (ww3_prnc) requires properly formatted input data, particularly regarding metadata and attributes.
If errors related to calendar or time units occur, the dataset may need to be corrected accordingly. 
```
ncatted -h -a calendar,time,c,c,'standard' wind.nc
ncatted -h -a units,time,o,c,'hours since 1900-01-01T00:00:00Z' wind.nc
```
**note**: you should always check after the above modifications, if the dates of your data file are correct.
```
cdo sinfo wind.nc
```
## Reordering latitude and longitude dimensions
If the data coordinates are not aligned with the model grid, they can be rearranged as needed.
```
ncpdq -h -O -a -lat ERA5_forcing.nc ERA5_forcing.nc
```
This is typically required when variables are expected in the form (time, lat, lon) but are instead provided as (time, lon, lat).

## Handling missing _FillValue

WaveWatch III requires a fill value for points outside the model domain (e.g., land points).
- Via cdo
```
cdo -setattribute,10u@_FillValue=-32767s \
    -setattribute,10v@_FillValue=-32767s \
    ERA5_forcing.nc ERA5_forcing.nc_

mv ERA5_forcing.nc_ ERA5_forcing.nc
```
- Via nco

```
ncatted -O \
  -a _FillValue,10u,a,c,-32767s \
  -a _FillValue,10v,a,c,-32767s \
  file1.nc file2.nc
```
## Change variable name
Change name of a variable from ```temp``` to ```t2m```
```
cdo -chname,temp,t2m infile outfile
```
## Count zeros in a specific variable
```
ncap2 -O -s 'zero_count=(my_variable == 0).total()' infile.nc count.nc
ncdump -h count.nc
```
Useful for identifying zeros inside your data, which may lead to some infinities in model's computations. If that is the case, you can simply set _FillValue to zero to mask those points.

