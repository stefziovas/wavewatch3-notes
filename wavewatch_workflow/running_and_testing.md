# Running WaveWatch III & Test Cases

## 1) Standard execution order

1) ww3_grid – grid setup

2) ww3_strt – initial conditions

3) ww3_prnc – prepare forcing

4) ww3_shel – main integration

5) ww3_ounf – full-grid output

6) ww3_ounp – point output

Namelists are located in model/nml.

## 2) Running test cases

Navigate to the test case directory

Read instructions:

cat info

Run run_test with selected options

Outputs appear in the work directory

3) Enabling NetCDF output in test cases
export WWATCH3_NETCDF=NC4
export NETCDF_CONFIG=/apps/NETCDF-FORTRAN/4.5.3/bin/nf-confi
