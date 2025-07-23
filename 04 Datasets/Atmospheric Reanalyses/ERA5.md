# ERA5

[Homepage](https://www.ecmwf.int/en/forecasts/datasets/reanalysis-datasets/era5)  
Download location: [Copernicus](https://cds.climate.copernicus.eu/#!/search?text=ERA5&type=dataset)  

**Download procedure** ([info](https://confluence.ecmwf.int/display/CKB/How+to+download+ERA5); [YouTube video series](https://www.youtube.com/channel/UC94xkaJn1NkxR4trAfVArbg)):
* Request fields, region, and timespan at download location
* Download locally, **or** use [CDSAPI, python-based approach](https://cds.climate.copernicus.eu/api-how-to).
    * [YouTube tutorial for CDSAPI](https://www.youtube.com/watch?v=H47VBPeUGQo)
    * [Example of CDSAPI python script with `for` loop](https://hackmd.io/ENRMLQIeQJi847OIFXA4tw)
    * Use online selection tool at download location to check variable names, as well as header info and product type for specific dataset.
* Option to download GRIB file and convert to NetCDF ([how to](https://confluence.ecmwf.int/display/OIFS/How+to+convert+GRIB+to+netCDF); simplest approach is using [`cdo`](https://code.mpimet.mpg.de/projects/cdo)) **OR** download directly as netcdf (preferred)
* A downloadable grid file is not available for ERA5. See [here](https://confluence.ecmwf.int/display/CKB/ERA5%3A+What+is+the+spatial+reference) for information about the grid. Code to evaluate area for a regular lat/lon grid is available in my `utils` package [here](https://github.com/gmacgilchrist/utils) (in the `geo` module).

## Notes

### Differing timestamps across variables
Downloading ERA5 monthly data, heat fluxes from the ocean surface have a timestamp at 0600 whereas state variables like air and surface have a timestamp at 0000. A preprocess step using the `floor` category of the datetime accessor in `xarray` allows them to be loaded together:
```
import xarray as xr
basedir = '/gws/nopw/j04/co2clim/datasets/ERA5/monthly/'
filename = 'era5.1x1.monthly.*.*.nc'
def preprocess(ds):
    time = ds.valid_time.dt.floor("D")
    return ds.assign_coords(valid_time=time)
ds = xr.open_mfdataset(basedir+filename,preprocess=preprocess)
```