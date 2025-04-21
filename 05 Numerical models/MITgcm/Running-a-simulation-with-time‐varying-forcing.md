# Time varying forcing

# Writing a time-varying forcing file

Write your time-varying forcing as a 3D array with dimensions nTime x ny x nx and save it to a *.bin file. For an example, see script “genbin_TimeVarExample10yr_Tair.py”. The time dimension can be whatever resolution you like (yearly, decadal, etc.). 

Note, per the odd behaviour documented below, Maddie recommends writing a *.bin file covering the full length of time of your planned simulation, plus ~10 timesteps. 

## Cyclical forcings – un-tested,  not verified!

If you have periodic/repeating forcing, you can write a *.bin file covering only one cycle, and MITgcm will  know to loop over this *.bin file once it reaches its end (e.g., if you want T_air forcing with a period of 500 years, you can write a T_air.bin file covering one full cycle/500 years and then run a simulation for e.g. 1000 years, and MITgcm will know to return to the start of the T_air.bin file once it reaches its end). 

In theory, all you have to do set “repeatPeriod” under &EXF_NML_01 in the data.exf file to the period of the forcing in seconds. This (I believe) will tell the model to cycle through all the input fields (*.bin files) after this number of seconds (all input fields, that is, that have a non-zero period under &EXF_NML_02 in data.exf. All 2D (non-time-varying) input fields should have their respective periods set to “0.0,”, and this should tell the model that this “repeatPeriod” parameter does not apply to them.) 

However, Maddie has not tested this and someone else should! Per the odd behavior documented below, Maddie recommends writing a long, repeating *.bin file covering the full length of time of your planned simulation, plus ~10 timesteps.


# !! Odd behaviour: Add ~10 extra timesteps to your forcing file

The T_air.bin file produced by “genbin_TimeVarExample10yr_Tair.py” is of yearly resolution and covers 11 years ([nT x ny x nx] = [11, 140, 72]), with 2m air temperature going from 1, 2, 3… up to 11 degC each year. The model does not behave as expected for a 10-year long simulation, but does behave as expected for a 5-year long resolution. 

Compare the output of run_test.TimeVarExample05yr versus run_test.TimeVarExample10yr (in /work/e786/e786/shared/mshankle/MITgcm/verification/MS2025/ on ARCHER2). These simulations provide output every 0.5 years over a 5- and 10-year simulation, respectively. (Maddie has made a start on this – ask for her “validating__run_test.TimeVarExample10yr.xlsx” file.) 

In the 5-year simulation, temperature starts at 1 degC and progresses each year as expected. However, in the 10-year simulation, it appears to “reach backwards” and interpolate between the last (11 degC) and first (1 degC) entry in T_air.bin when forcing the first timestep (temperature = ~10.75 degC).
This appears to relate to how MITgcm time-averages its output. Perhaps it takes the first entry of T_air.bin but averages “back in time” (e.g. the 10.75 degC value makes it seem like the model is “reaching backwards” from its first entry and averaging over the 6 months leading up to that point, i.e., last 6 months represented by the T_air.bin file which cover a temperature range of 10.5-11 degC. Or something like that…) See any output’s *.meta file to investigate what time period it’s averaging over.  

**To do:** investigate and fully understand this behaviour. 

**Recommended action for now:** write *.bin forcing files that are ~10 timesteps (just to be safe) longer than the full simulation time you plan to run. 


# Set parameters in data.cal and data.pkg

FYI (no clear documentation on this, gathered from correspondence on MITgcm mailing list), data.cal's "startDate_1" corresponds to the date (YYYYMMDD) that should be represented by iteration number 0 (NOT the date represented by nIter0 in a given run). So, it should usually be set to “=00010101,” (Note, it cannot take 0000-01-01 or 0000-00-00 as input. So, the iteration number on any given output file represents the run time SINCE year 0001, so e.g. if iterNum*deltaT = 24500 years, then technically the date of this output is 245001-01-01, not 24500-01-01.) “startDate2” refers to HHMMSS and can usually be left as “000000”, and “TheCalendar=’model’,” instructs MITgcm to use the default calendar, which is a 360-day year. 

Ensure the EXF package is turned on a run time with “useEXF = .TRUE.” in data.pkg. 


# Set parameters in the data.exf file

Here I will clarify some parameters that are not well described in the [documentation](https://mitgcm.readthedocs.io/en/latest/phys_pkgs/exf.html#general-flags-and-parameters). 

First, under &EXF_NML_01, “repeatPeriod” should be set to 0.0, instructing the model to use the respective periods assigned to each input field under &EXF_NML_02 (unless you are testing out cyclical forcing – see section “Cyclical forcings – un-tested,  not verified!”, under “Writing a time-varying forcing file” above). 

Then, under &EXF_NML_02, each forcing (*.bin) files will have three parameters associated with it that are set in this file: its “startdate1”, “startdate2”, and “period”. These parameters will only matter and be used for forcing files which are three-dimensional (which will be indicated by a non-zero “period” parameter). 

_**For any forcing file that is CONSTANT in time**_ (i.e., not time-varying and two-dimensional, nY x nX): “startdate1” should equal “00010101,” and its “period” should equal “0.0,” and its “startdate2” should be commented out, as below e.g. (the 180000 below is left over from a seasonal experiment from Marzocchi & Jansen 2019 (Nature Geoscience)): 
```
precipstartdate1  = 00010101,
# precipstartdate2  = 180000,
precipperiod      = 0.0,
```

_**For any forcing file that is TIME-VARYING**_ (i.e., three-dimensional, nT x nY  nX): edit the three parameters appropriately: 

`atempstartdate1`   = specifies the date (YYYYMMDD or YYYYYMMDD) that the first entry of this forcing file represents. This will correspond to the date given by nIter0 of your run, plus one year (see explanation in section “Set parameters in data.cal and data.pkg”). If you have written a time-varying file that will cover more than one model simulation (e.g., a 10,000-year long T_air.bin file that will be used over n=10 1000-year long runs), you only need to set this startdate as the nIter0 for your very first run. It will not need to be updated on subsequent runs (nothing in this data.exf file will), and the model will know where to start reading T_air.bin for each subsequent run given that run’s nIter0 number and the start date given here. 

`atempstartdate2`   = specifies the time (HHMMSS) that the first entry of this forcing file represents (but we typically don’t use this and leave it commented out) 

`atempperiod`   = this is not “period” in the cyclical sense, but better thought of as the resolution of your forcing file (i.e., the interval in seconds between two entries in your forcing file) 

So for example, the parameters for a time-varying T_air.bin file might look like: 
```
atempstartdate1   = 300010101,
# atempstartdate2   = 180000,
atempperiod       = 311040000.0,
```
The above settings indicate that the first entry in the forcing file represents 01 January 30,000 and that each entry in the forcing file is 10 years apart (“period = 311040000 seconds /60/60/24/360 = 10 years). Note that MITgcm uses a 360-day calendar because we have set the model to use the default calendar (TheCalendar=’model’, in the “data.cal” file), which is a 360-day calendar ([documentation](https://mitgcm.readthedocs.io/en/latest/phys_pkgs/cal.html#the-individual-calendars)).  


# Edit the data.diagnostics file if you want to output the forcing itself as a diagnostic 

Any of your forcing fields (time-varying or not) can be put out as a diagnostic by the model if you include them in your data.diagnostics file. See [this](https://mitgcm.readthedocs.io/en/latest/phys_pkgs/exf.html#exf-diagnostics) page for the list of variable names and below for example entries in the data.diagnostics file: 
```
&DIAGNOSTICS_LIST
  fileName(1) = 'DiagEXF',
  fields(1,1) = 'EXFatemp',
  frequency(1) = 15552000,

  fileName(8) = 'DiagEXFSnap',
  fields(1,8)  = 'EXFatemp',
  frequency(8) = -3110400000,
  timePhase(8) = 0,
```

The code above instructs MITgcm to write diagnostic files titled  'DiagEXF' and 'DiagEXFSnap', each with a single variable ('EXFatemp') which will give the field of 2m air temperatures specified in your T_air.bin forcing file at the “frequency” (every 100 years, or 3110400000 seconds). The negative frequency of DiagEXFSnap instructs the model to output a “snapshot” of T_air at every 100 years exactly, while the positive frequency of DiagEXF will produce a time-average of T_air, averaged over every 100 years (I believe it is averaged over the preceding 100 years, but you can check this in the “timeInterval” parameter given in any output’s *.meta file) (I also believe the interval over which time-averaged output is averaged can be altered with a “timePhase” parameter, though Maddie has only used this parameter for snapshot output and has never tested whether it works on time-averaged output). See the top of the data.diagnostics file for the documentation on “frequency”. 

!! _**Note, it is important to specify the timePhase as 0 for snapshot diagnostics**_ because according to section [“9.1.4.2.2 Using pkg/diagnostics for adjoint variables”](https://mitgcm.readthedocs.io/en/latest/outp_pkgs/outp_pkgs.html#using-pkg-diagnostics-for-adjoint-variables) : 

_“Note: the diagnostics package automatically provides a phase shift of frequency/2, so specify timePhase = 0 to match output from adjDumpFreq.”_ 

Setting timePhase = 0 will write output at time = timePhase + multiple of |frequency| (according to documentation at the top of data.diagnostics file). If timePhase = frequency/2, you will get snapshot output not at every 100 years (0, 100, 200, …) but snapshots at 50, 150, 250, etc. So, if you want snapshots at 0, 100, 200, specify timePhase as 0. 

**_It is unclear whether this timePhase parameter should also be used for non-snapshot (i.e., time-averaged) output._** See section “!! Odd behaviour: Add ~10 extra timesteps to your forcing file” above for a description of odd time-averaging behaviour that needs to be investigated, understood, and documented.

  
# Run simulation as normal

From here, you can start your simulation as normal. 
