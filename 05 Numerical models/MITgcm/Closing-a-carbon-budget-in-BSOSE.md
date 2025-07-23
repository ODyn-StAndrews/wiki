# Closing budgets in MITgcm
*Contributors*: [Iris Liang](https://github.com/xliang576)

## What is "closing a budget" and why use budgets
A lot of processes are programmed and calculated inside the model. When the model runs, they collectively update the model state (temperature, nutrients, dic concentration, etc.). How to restore the contribution of each processes? That is what budgets do. "Closing a budget" means attributing the changes of some model states to the known/coded processes responsible as time proceeds, within a neglectable residual range. In the end, we will be able to get a time series showing how each process acts on the tracer field. 

To help build the budgets and do analysis, we store and output [diagnostics](https://mitgcm.readthedocs.io/en/latest/outp_pkgs/outp_pkgs.html). These are additional calculated variables derived as the model runs. In this way the ocean state and processes are computationally consistent - an advantage over observations.

## Links and resources
- Documentation (from 2014) on heat and salt budgets (with linear free surface and virtual freshwater flux): [link](https://mitgcm.org/download/daily_snapshot/MITgcm/doc/Heat_Salt_Budget_MITgcm.pdf)
    - The above documentation arose from a discussion on the MITgcm-support email thread: [link to start of discussion](http://mailman.mitgcm.org/pipermail/mitgcm-support/2014-November/009586.html) (click through "Next message" to follow discussion)
- Documentation (from 2017) on general budgets (with nonlinear free surface and real freshwater fluxes) with specific example for ECCOv4r3: [link](https://dspace.mit.edu/handle/1721.1/111094)

## Closing the carbon budget in B-SOSE
Since BSOSE uses data assimilation by reversingly adjusting boundary conditions (without adding or removing unsourced properties), all the budgets *should* be able to close, unless tricky things happen when the model diagnostics are generated or stored. Intricacies also arise from free surface and vertical coordinates. Here are [some BSOSE biogeochemical budget examples](https://sose.ucsd.edu/budgets/).

Lists of available diagnostics are available for most of the BSOSE iterations, e.g. for [ITER155](https://sose.ucsd.edu/SO6/ITER155/available_diagnostics.log). The "snapshots" diagnostics are snapshots at the end of 5-day/month, while others are time-averaged over the time period of 5-day/month.

### DIC transport equation
Tracer conservation equation is 
```math
\frac{\partial DIC}{\partial t} = - \vec{u} \cdot \nabla DIC + \gamma \nabla^2 DIC + \mathcal{F_a} + Bio
```
where $DIC$ is the concentration of DIC.

Written in Budget term is
```math
TEND =ADV + DIFF + SURF + BIO
```

### Terms in tracer budget
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

### Example code

🚧 Under construction 🚧