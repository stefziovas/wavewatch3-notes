# Workflows & Comparing Configurations

1) Standard execution order

1) ww3_grid – grid setup

2) ww3_strt – initial conditions

3) ww3_prnc – prepare forcing

4) ww3_shel – main integration

5) ww3_ounf – full-grid output

6) ww3_ounp – point output

Namelists are located in model/nml and in model/inp. If you have both inp and nml files of an executable in your working directory, then WW3 will use only nml.
For example, if you have ww3_prnc.nml and ww3_prnc.inp, then ww3_prnc will only read ww3_prnc.nml

## Running test cases

Navigate to the test case directory. Read instructions:
```
cat info
```
Run ```run_test``` with selected options. Outputs appear in the work directory. 

## Comparing different model configurations

1) Modify one parameter or switch

2) Change output filename in the test case or run

3) Compare results using:

- NetCDF analysis

- Python / MATLAB

- Spatial and temporal diagnostics

This avoids overwriting results and ensures reproducibility.
