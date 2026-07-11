# WaveWatch workflow

## Standard execution order

1) **ww3_grid** : Configuring the model grid (ww3_grid.nml) and essential simulation parameters—including time step, frequency, and directional ranges, while also calibrating physical parameterization coefficients and enabling additional features, like partitioning schemes and point spectral outputs (namelists.nml, points.nml).

- Inputs: 'grid file', 'obstr. file', 'mask file', ww3_grid.nml, namelists.nml, points.nml
- Outputs: 'mod def.ww3', 'mask.ww3', 'mapsta.ww3'

2) **ww3_strt** – Setting initial conditions (Gaussian, Jonswap and other several options for an initial frequency spectrum) 

- Inputs: 'mod def.ww3', ww3_strt.inp
- Outputs: 'restart.ww3'

3) **ww3_prnc** – Preparing model's forcing. WaveWatch requires at minimum wind forcing, while surface currents are also commonly used due to their significant impact on wave evolution.

- Inputs: 'mod def.ww3', '<wind_file>.nc', <currents_file>.nc, ww3_prnc.nml
- Outputs: 'wind.ww3', 'currents.ww3'

4) **ww3_shel** – This is the main executable of WaveWatch. Here, you can define the duration of the run by defining the start and the end date, decide which parameters are gonna been written out (in a WaveWatch binary format, which you can handle in post-process) and at what frequency, while also selecting which parameters the model gonna recieve from and sent back to other components in a coupled configuration (e.g. NEMO-WW3).   

- Inputs: 'mod def.ww3', 'restart.ww3', '<wind_file>.nc', <currents_file>.nc, ww3_shel.nml
- Outputs: 'out_grd.ww3', 'out_pnt.ww3', 'restartn.ww3'

5) **ww3_ounf** – Post-process executable for exporting fully-gridded outputs. It is not mandatory to export all the data produced, but rather you can define the dates for which you want to export data from the simulation, and also at what frequency (e.g. daily or every 3 hours). Although, has to be noted that the current WaveWatch code reads each time step of out_grd.ww3 till reaching the desired date, based on the output frequency you asked for in ww3_ounf.nml, instead of selecting as an index from the matrix of the desirable parameter (need development...). 

- Inputs: 'out_grd.ww3', ww3_ounf.nml
- Outputs: '<output_file_name>.nc'
  
6) **ww3_ounp** – Post-process executable for points outputs, that have been defined in points.nml file during ww3_grid. Opposite to ww3_ounf, ww3_ounp produces spectral data (E(f,θ)), instead of integrated physical parameters, like SWH and wave period.

- Inputs: 'out_pnt.ww3', ww3_ounp.nml
- Outputs: '<output_file_name>.nc'

Namelists are located in model/nml and in model/inp. If you have both inp and nml files of an executable in your working directory, then WW3 will use only nml.
For example, if you have ww3_prnc.nml and ww3_prnc.inp, then ww3_prnc will only read ww3_prnc.nml

## Splitting of the wave action equation

WaveWatch is builded around one main equation and that is the Wave action balance equation, which describes the conservation of wave action. 

1) Update of water level
2) Intra-spectral calculations (part 1): Intergrated over ΔT_g/2
3) Spatial propagation: Intergrated over ΔT_g
4) Intra-spectral calculations (part 2): Intergrated over ΔT_g/2
5) Source term integration: integration over ∆T_g

**Note**: During WaveWatch runs, the cpus of a node are been distributed with a constant stride inside the model's domain, meaning that one grid point goona be managed by processor #1, the next grip point from processor #2, and so on till again a grid point falls becomes a responsibility of processor #1. This is happening to avoid overload inbalances, because imaging one cpu solving for the Aegean region, where it is a sunny day, and another core solves for the Iberian region in a windy day, then the Iberian core would be significantly slower than the other processor. 

Anyhow, the important here is that each processor handles bunch of different grid points, where it solves the wave action balance equation locally for those grid points. These local computations refers to the spectrum "propagation" of the wave action or in other words the interactions between the different frequencies of a wave pulse. Then, when the spatial propagation step arrives, WaveWatch gathers all data from all grid points for each specific spectral point (a combination of a specific frequency and direction) to only one processor. During this step, each processor solves the global propagation matrix for the specific wave component it is responsible for. After that each processor return the spatial updated data to their mother processor, and finally in the next step source terms are applied. During this process of gathering data from all the other cpus, each processor needs to "put aside" each local spectral computations and data. This requires free space to read and write new data into memory for the spatial propagation step, in order to avoid missing local data, reaching to other cpus cache or fighting for RAM. The most optional solution is to use a hybrid MPI-OPENMP configuration, where each MPI processor gonna have a team of several OPENMP physical cores (NOT multithreading). In this set up, all cores inside the team are sharing both tasks and memory. This happens during local spectral calculations, but during spatial propagation the OPENMP cores are waiting, till the MPI core of the "team" is done with the communication with other MPI cores and with the computations of spatial propagation. Note though that via this setup, the MPI core gonna have more free memory space to touch during the spatial propagation step.

<img width="937" height="465" alt="1000003087" src="https://github.com/user-attachments/assets/8df8b5aa-ef9f-4799-a7cc-b66d1050312c" />
 
## Running test cases

Navigate to the test case directory. Read instructions:
```
cat info
```
Run ```run_test``` with selected options. Outputs appear in the work directory. 

## Example of tuning physical parameterization coefficients in namelists.nml 
```
&SIN4
  BETAMAX = 1.55,
  Z0MAX = 0.002,
  TAUWSHELTER = 0.0,
  SWELLF3 = 0.015,
  SWELLF4 = 1.0E5,
  SWELLF7 = 0.0
/

&SDS4
  SDSBCHOICE = 1,
  FXFM3 = 2.5,
  SDSBR = 0.00085,
  SDSCUM = 0.0,
  SDSCOS = 0.0
/

&SNL1
  NLPROP = 2.7E7
/

&MISC
  STDX = 11.2,
  STDY = 11.2,
  STDT = 1800.,
  FLAGTR = 4
/
```
These parameters control: wind input, dissipation, nonlinear interactions, time stepping and diagnostics
