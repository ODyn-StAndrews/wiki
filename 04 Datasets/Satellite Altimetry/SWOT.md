# SWOT 

[Homepage](https://swot.jpl.nasa.gov/) 

Download location: [NASA PO.DAAC](https://podaac.jpl.nasa.gov/SWOT?tab=mission-objectives&sections=about%2Bdata) and [AVISO](https://www.aviso.altimetry.fr/en/missions/current-missions/swot/access-to-data.html).

The **Surface Water and Ocean Topography** mission provides sea surface height measurements with an unprecedented, submesoscale spatial resolution of 2km x 2km thanks to an instrumented satellite launched on December 16, 2022.

**Download procedure** (for NASA PO.DAAC)
* An [Earthdata](https://urs.earthdata.nasa.gov/) login is needed.
* Follow the steps indicated [here](https://github.com/podaac/data-subscriber) to get the NASA PO.DAAC data downloader tool.
* When this is done, you can for example download the full SWOT SSH dataset since satellite launch with the following command: ```podaac-data-subscriber -c SWOT_L2_LR_SSH_D -d ./data/SWOT_L2_LR_SSH_D --start-date 2022-12-16T00:00:00Z```, as indicated [here](https://podaac.jpl.nasa.gov/dataset/SWOT_L2_LR_SSH_D).


