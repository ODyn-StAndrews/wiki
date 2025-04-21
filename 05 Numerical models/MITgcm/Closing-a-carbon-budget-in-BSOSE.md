# Closing budgets

## What is "closing a budget" and why use budgets
A lot of processes are programmed and calculated inside the model. When the model runs, they collectively update the model state (temperature, nutrients, dic concentration, etc.). How to restore the contribution of each processes? That is what budgets do. "Closing a budget" means attributing the changes of some model states to the known/coded processes responsible as time proceeds, within a neglectable residual range. In the end, we will be able to get a time series data showing how each processes act on the model domain. 

To help build the budgets and do analysis, the modellers store and output diagnostics. (See a full list of available [BSOSE diagnostics here](https://sose.ucsd.edu/SO6/ITER155/available_diagnostics.log)) These are additional calculated variables  when the model runs. In this way the ocean states and processes are computationally consistent - an advantage over observations. The "Snapshots" diagnostics are snapshots at the end of 5-day/month, while others are time-averaged over the time period of 5-day/month.

In this page the property focused on is dissolved inorganic carbon(DIC). 
Since BSOSE uses data assimilation by reversingly adjusting boundary conditions (without adding or removing unsourced properties), 
all the budgets *should* be able to close, unless tricky things happen when the model diagnostics are generated or stored. Intricacies also arise from free surface and vertical coordinates.
Here are [some BSOSE biogeochemical budget examples](https://sose.ucsd.edu/budgets/).

## DIC transport equation
Tracer conservation equation is 
```math
\frac{\partial DIC}{\partial t} = - \vec{u} \cdot \nabla DIC + \gamma \nabla^2 DIC + \mathcal{F_a} + Bio
```
where $DIC$ is the concentration of DIC.

Written in Budget term is
```math
TEND =ADV + DIFF + SURF + BIO
```

## Terms in tracer budget
LHS $`TEND`$ is the time difference of concentration `TRAC01`, namely
```math 
\frac{DIC(t_{n+1}) - DIC(t_n)}{\Delta t} 
```

[(?) Or with correction for nonlinear surface in BSOSE.
```math
\frac{DIC(t_{n+1})*(1+h(t_{n+1}) )/H-DIC(t_n)*(1+h(t_{n}) )/H}{\Delta t} 
```
where $`h`$ is sea surface height anomaly (diagnostics`ETAN`), $`H`$ is ocean depth. (?)]



The corresponding process and diagnostics for RHS are given in the following table.

|   Term	|   Process	|   Diagnostics	|  
|---	|---	|---	|  
|   ADV	|   Advective transport	|   `ADVxTr01`,`ADVyTr01`,`ADVrTr01`|  
|   DIFF	|   Diffusive transport	|   `DFxETr01`,`DFyETr01`,`DFrITr01`	|  
|   SURF	|   Surface sea-air carbon flux	|   `BLGCFLX`	|   
|   BIO	|   Biological source and sinks	|   `BLGBIOC`	|   

## Example code

See repository xxx. 

## Help on other budgets and budget analysis
Guideline on heat, salinity and other budget (in MITgcm) can be found [here](https://mitgcm.org/download/daily_snapshot/MITgcm/doc/Heat_Salt_Budget_MITgcm.pdf),
[here](https://www.ecco-group.org/docs/evaluating_budgets_in_eccov4r3_updated_20220118.pdf), 
and (in MOM6) [here](https://mom6-analysiscookbook.readthedocs.io/en/latest/notebooks/Closing_tracer_budgets.html).

Some analysis on BSOSE budget can be found [here](https://github.com/gmacgilchrist/bsose/tree/main).