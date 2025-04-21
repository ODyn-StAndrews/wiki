# MERRA2

[Homepage](https://gmao.gsfc.nasa.gov/reanalysis/MERRA-2/)  
[NCAR Climate Data Guide page](https://climatedataguide.ucar.edu/climate-data/nasas-merra2-reanalysis)  
Download location: [GES DISC DAAC](https://disc.gsfc.nasa.gov/datasets?project=MERRA-2)  

**Download procedure**
* Make sure you have a [NASA EarthData](https://www.earthdata.nasa.gov/) account.
* Follow the instructions [here](https://disc.gsfc.nasa.gov/data-access) to set up authorization for `wget` (this includes creating a `.netrc` with login credentials and `.urs_cookies` file).
* From the Homepage, or Climate Data Guide, navigate to Data Access/Get Data. This will redirect to NASA's [GES DISC DAAC](https://disc.gsfc.nasa.gov/datasets?project=MERRA-2) (link might break).
* Select required data (see examples for ocean surface diagnostics below) and navigate to the online archive.
* Use the URL for the online archive with a `wget` command to retrieve the files of interest. Examples of different `wget` options are given [here](https://disc.gsfc.nasa.gov/information/howto?title=How%20to%20Download%20Data%20Files%20from%20HTTPS%20Service%20with%20wget); I used the multiple file download with wildcards under **Section 7** of that documentation.

*DAAC summary pages for ocean surface data*
* [Surface ocean diagnostics, monthly means (M2TMNXOCN)](https://disc.gsfc.nasa.gov/datasets/M2TMNXOCN_5.12.4/summary)
* [Surface flux diagnostics, monthly means (M2TMNXFLX)](https://disc.gsfc.nasa.gov/datasets/M2TMNXFLX_5.12.4/summary)
* [Radiation diagnostics, monthly means (M2TMNXRAD)](https://disc.gsfc.nasa.gov/datasets/M2TMNXRAD_5.12.4/summary)

*Example `wget` command*
```wget --load-cookies ~/.urs_cookies --save-cookies ~/.urs_cookies --keep-session-cookies -r -c -nH -nd -np -A nc4,xml  --content-disposition "https://goldsmr4.gesdisc.eosdis.nasa.gov/data/MERRA2_MONTHLY/M2TMNXFLX.5.12.4/"```
where
```--load-cookies ~/.urs_cookies --save-cookies ~/.urs_cookies --keep-session-cookies``` are the required authorization flags. This downloads the surface flux diagnostic dataset (M2TMNXFLX) for all years.