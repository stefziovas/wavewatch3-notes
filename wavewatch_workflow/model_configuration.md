# Model Configuration & Namelists

## 1) Source term parameterization (ww3_grid namelist)

Example configuration:

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

These parameters control:

wind input, dissipation, nonlinear interactions, time stepping and diagnostics

## 2) Selecting a model switch

./w3_setup ~/WW3-6.07.1/model/ -c ionio -s Ifremer1

Then compile selected programs:

./w3_make ww3_grid ww3_prnc ww3_strt ww3_shel ww3_ounf ww3_ounp
