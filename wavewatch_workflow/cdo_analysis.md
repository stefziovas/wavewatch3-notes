# CDO-Based Analysis for WaveWatch III Output

## Computing anomalies

Example: significant wave height anomalies

- Step 1 – Extract variable
```
cdo select,name=hs ww3.run_no_currents.nc ww3.hs.nc
```
- Step 2 – Compute time mean (climatology)
```
cdo timavg ww3.hs.nc ww3.hs_mean.nc
```
- Step 3 – Compute anomalies
```
cdo sub ww3.hs.nc ww3.hs_mean.nc ww3.hs_anomalies.nc
```
## Seasonal statistics

- Manual seasonal aggregation (example: MAM 2020)
```
cdo mergetime hs_2020_03.nc hs_2020_04.nc hs_2020_05.nc hs_MAM_2020.nc
cdo timmean hs_MAM_2020.nc hs_MAM_mean_2020.nc
```
- Seasonal selection from annual file (merge in time monthly data into a yearly file)
```
cdo mergetime hs_2020_01.nc hs_2020_02.nc ... hs_2020_12.nc hs_2020.nc
```
- Select season (e.g. JFM)
```
cdo select,season=JFM hs_2020.nc hs_JFM.nc
cdo timmean hs_JFM.nc hs_JFM_mean.nc
cdo timstd  hs_JFM.nc hs_JFM_std.nc
```
- Merge statistics into one file
```
ncks -A hs_JFM_mean.nc hs_JFM_std.nc
mv hs_JFM_std.nc hs_JFM_stats.nc
```

## Monthly statistics

- Monthly mean from daily output
```
cdo select,name=hs ww3.run_no_currents.20200101.nc ww3.hs_jan.nc
cdo timmean ww3.hs_jan.nc ww3.hs_jan_mean.nc
```
- Difference between two experiments (currents vs no currents)
```
cdo timmean -select,name=hs ww3.run_no_currents.20200101.nc ww3.hs_no_cur_mean_01.nc
cdo timmean -select,name=hs ww3.run_with_currents.20200101.nc ww3.hs_with_cur_mean_01.nc
```
Compute differences: 
```
cdo sub ww3.hs_with_cur_mean_01.nc ww3.hs_no_cur_mean_01.nc ww3.diff_mean_01.nc
```
## Adding variables to an existing NetCDF file

Purpose: Combine multiple statistics into a single output file.
```
ncks -A ww3.hs_mean.nc ww3.hs_std.nc
mv ww3.hs_std.nc ww3.hs_stats.nc
```
## Computing a normalized index

Example: relative impact of currents on wave height
```
cdo sub ww3.hs_run_with_currents.nc ww3.hs_run_no_currents.nc ww3.diff.nc
cdo div ww3.diff.nc ww3.hs_run_no_currents.nc index.nc
```
This produces a dimensionless index expressing relative change.

## Computing derived variables using exprf

- Computing wind speed and direction (context of exprf file: wind_exprf.txt)
```
speed = sqrt(uwnd^2 + vwnd^2);
dir   = atan(vwnd / uwnd);
Apply expressions
cdo exprf,wind_exprf.txt ww3.wind.nc ww3.windchar.nc
ncks -A ww3.windchar.nc ww3.wind.nc
```
⚠️ Note
atan2(x,y) does not work reliably inside exprf files, maybe due to the version of cdo. 

- Wind–current interaction metrics (context of exprf file: wnd_cur_exprf.txt)
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
