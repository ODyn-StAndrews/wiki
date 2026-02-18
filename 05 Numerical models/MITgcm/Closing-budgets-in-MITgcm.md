# Closing budgets in MITgcm
*Contributors*: [Iris Liang](https://github.com/xliang576)

## What is "closing a budget" and why use budgets
A lot of processes are programmed and calculated inside the model. When the model runs, they collectively update the model state (temperature, nutrients, dic concentration, etc.). How to restore the contribution of each processes? That is what budgets do. "Closing a budget" means attributing the changes of some model states to the known/coded processes responsible as time proceeds, within a neglectable residual range. In the end, we will be able to get a time series showing how each process acts on the tracer field. 

To help build the budgets and do analysis, we store and output [diagnostics](https://mitgcm.readthedocs.io/en/latest/outp_pkgs/outp_pkgs.html). These are additional calculated variables derived as the model runs. In this way the ocean state and processes are computationally consistent - an advantage over observations.

## Links and resources
- Documentation (from 2014) on heat and salt budgets (with linear free surface and virtual freshwater flux): [link](https://mitgcm.org/download/daily_snapshot/MITgcm/doc/Heat_Salt_Budget_MITgcm.pdf)
    - The above documentation arose from a discussion on the MITgcm-support email thread: [link to start of discussion](http://mailman.mitgcm.org/pipermail/mitgcm-support/2014-November/009586.html) (click through "Next message" to follow discussion)
- Documentation (from 2017) on general budgets (with nonlinear free surface and real freshwater fluxes) with specific example for ECCOv4r3: [link](https://dspace.mit.edu/handle/1721.1/111094)
- An introductory slide on the application of budgets (in ECCO): [link](https://ecco-group.org/docs/23_Pillar_ECCO_Budgets.pdf)
- ECCO tutorials on budget closure for [volume](https://ecco-v4-python-tutorial.readthedocs.io/ECCO_v4_Volume_budget_closure.html), for [heat](https://ecco-v4-python-tutorial.readthedocs.io/ECCO_v4_Heat_budget_closure.html), for [salt, salinity and freshwater](https://ecco-v4-python-tutorial.readthedocs.io/ECCO_v4_Salt_and_salinity_budget.html) 
- An example notebook of dissolved carbon budget in BSOSE: [link](https://odyn-standrews.github.io/wiki/closing-the-carbon-budget-in-b-sose/)


