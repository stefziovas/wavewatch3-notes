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
If errors related to calendar or time units occur, the dataset may need to be corrected accordingly. The typical error that WW3 shows when the time format of a forcing is off, is the following:
```
*** WAVEWATCH III ERROR IN W3TIMEMD :
PREMATURE END OF TIME ATTRIBUTE
hours since 2001-12-07 00:00:00
DIFFERS FROM CONVENTIONS ISO8601
XXX since YYYY-MM-DD hh:mm:ss
XXX since YYYY-M-D h:m:s
XXX since YYYY-M-D hh:mm:ss
```
In order to fix it, we usually use something like the following two commands:

```
ncatted -h -a calendar,time,c,c,'standard' wind.nc
ncatted -h -a units,time,o,c,'hours since 1900-01-01T00:00:00Z' wind.nc
```
**note**: you should always check after the above modifications, if the dates of your data file are correct.
```
cdo sinfo wind.nc
```

**Caution**: When you type ```ncatted -h -a units,time,o,c,'seconds since 1970-01-01T00:00:00Z' wind.nc```, everything works good and ww3_prnc run without a problem. But, when you change seconds in hours (```ncatted -h -a units,time,o,c,'hours since 1970-01-01T00:00:00Z' wind.nc```), you will get non sense dates in the ```cdo sinfo wind.nc```, like  ```116925-12-15 00:00:00``` 

Why is this happening and how this command actually works, so you can use it properly ?

**Anwser**: The reason you got that futuristic date is because ncatted is just a text editor. It only changes the label (the attribute) and does not touch the actual numbers stored inside the file. Let's assume the actual numeric value stored in your time variable is 1,000,000,000.

- First case (seconds):
1. The stored number is 1,000,000,000.
2. CDO reads your label: ```seconds since 1970```
3. It does the math: 1,000,000,000 \60 \60 \24 ~ 11,574 days.
4. It adds 11,574 days to the year 1970, correctly putting you in the year 2001.

- Second case (hours):
1. The stored number is still exactly 1,000,000,000.
2. But you changed the label to: ```hours since 1970```
3. Now CDO thinks that number represents hours!
4. It does the math: 1,000,000,000 \24 ~ 41,666,666 days.
5. It adds 41.6 million days to the year 1970, launching you straight to the year 116925!

NetCDF files do not store dates as calendar strings like ```2001-12-07```. Instead, they store two separate things:
a) The Values (Numbers): A list of pure numbers (e.g., 100, 101, 102).
b) The Units (Attributes): A text string acting as a rule (e.g., hours since 1970-01-01).

When you open a netcdf with ```CDO```, it takes the numbers, applies the text rule as a mathematical formula, and calculates the human-readable date on the fly.
With ```ncatted``` you are only changing the text rule. If your numbers were calculated as seconds, and you relabel them as hours, the dates will break.

If you want to switch your file from seconds to hours without breaking the dates, you must divide the numeric time values by 3600 AND change the label. You can do both in a single step using the ncap2 tool :

```ncap2 -s 'time=time/3600' -s 'time@units="hours since 1970-01-01T00:00:00Z"' test.nc test_hours.nc```

**Best practice** 

First you check the time format of the data you working with via cdo sinfo (e.g. ```cdo sinfo wind.nc --> RefTime =  1970-01-01 00:00:00  Units = seconds  Calendar = proleptic_gregorian```). After that, for safety reasons, you copy your data file to a ```test.nc``` file, and then, you adjust your ncatted command to the units of your working file (e.g. ```ncatted -h -a units,time,o,c,'seconds since 1970-01-01T00:00:00Z' test.nc```) or if you want to change the time label, for example from seconds to hours, you do: ```ncap2 -s 'time=time/3600' -s 'time@units="hours since 1970-01-01T00:00:00Z"' test.nc test_hours.nc```

***note***: it is no need to change the reference time of your data to the actual date (e.g. the first date of your data, e.g. 07/12/2001). The netcdf is formated in that way, in which ww3_prnc read the actual date of your data.

***About the actual commands***

- The command ```ncatted -h -a units,time,o,c,'seconds since 1970-01-01T00:00:00Z' test.nc``` is used to overwrite or create the units attribute for the time variable in a NetCDF file.Here is the breakdown of each part of the command:
1. ```ncatted```: This is the netCDF Attribute Editor from the NCO (netCDF Operators) suite.
2. ```-h```: Prevents the command from adding itself to the global history attribute of the NetCDF file.
3. ```-a```: Flag indicating that an attribute modification follows. The arguments following it are typically structured as att_nm,var_nm,mode,att_typ,att_val.
4. ```units```: The name of the attribute being modified (e.g., units, long_name).
5. ```time```: The name of the variable to which the attribute belongs.
6. ```o``` (mode): Specifies the overwrite mode. If the attribute already exists, it is updated; if not, it is created.
7. ```c``` (type): Specifies the attribute type as character (string).
8. ```'seconds since 1970-01-01T00:00:00Z'```: The actual value being assigned to the units attribute.
9. ```test.nc```: The target input NetCDF file to be modified

- The command ```ncap2 -s 'time=time/3600' -s 'time@units="hours since 1970-01-01T00:00:00Z"' test.nc test_hours.nc``` converts the time variable from seconds to hours and updates the corresponding units attribute.Here is the breakdown of each part of the command:
1. ```ncap2```: This is the netCDF Arithmetic Processor from the NCO suite, used for processing and manipulating data with scripts.
2. ```-s```: The script flag. It tells the operator that the following quoted string is a command to be executed. You can use multiple -s flags to chain operations.
3. ```'time=time/3600'```: The first operation. It divides every value in the time variable by \(3600\) (converting seconds to hours) and overwrites the time variable with these new values.
4. ```'time@units="hours since 1970-01-01T00:00:00Z'"```: The second operation. The @ symbol denotes an attribute in ncap2. This line changes the units attribute of the time variable to match the new hourly data.
5. ```test.nc```: The input NetCDF file containing the original data.
6. ```test_hours.nc```: The output NetCDF file where the modified data will be saved.

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

