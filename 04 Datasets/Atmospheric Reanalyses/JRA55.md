# JRA55

[Homepage](https://jra.kishou.go.jp/JRA-55/index_en.html)  
[User Guide](https://rda.ucar.edu/datasets/ds628.0/docs/JRA-55.handbook_TL319_en.pdf)  
Download location: [NCAR monthly mean](https://rda.ucar.edu/datasets/ds628.1/#!description)  
[List of variable names and numbers](https://rda.ucar.edu/datasets/ds628.0/#metadata/grib.html?_do=y) (This link is for 3 and 6-hourly data, but has the same variable name:number pair for reference)

**Download procedure**:
* Globus file transfer
    * Collection: NCAR RDA Dataset Archive
    * Path: `/ds628.1/`
* Files downloaded as GRIB, convert to NetCDF ([how to](https://confluence.ecmwf.int/display/OIFS/How+to+convert+GRIB+to+netCDF); simplest approach is using [`cdo`](https://code.mpimet.mpg.de/projects/cdo))
    * Shell script to perform conversion for all files (example available [here](https://hackmd.io/i3bvobigQlKZZRchI8OLYQ))
* Grid variables (e.g. cell area) are apparently not available from the NCAR website. The [input4MIPs website](https://esgf-node.llnl.gov/search/input4mips/) has `areacella` for the JRA55-do data product (search 'JRA55' in the search bar). I *think* this is the same grid as for the atmospheric reanalysis data.
    * Command to retrieve this file:
    * ```wget https://esgf-data2.llnl.gov/thredds/fileServer/user_pub_work/input4MIPs/CMIP6/OMIP/MRI/MRI-JRA55-do-1-5-0/atmos/fx/areacella/gr/v20200916/areacella_input4MIPs_atmosphericState_OMIP_MRI-JRA55-do-1-5-0_gr.nc```

* **NOTE**: variables downloaded from the NCAR Archive have their latitude coordinate *reversed*; that is, going from north to south. This is different to the grid data downloaded from input4MIPs. This can be remedied in loading the data using xarray's `.sort` on the latitude dimension.


An alternative download strategy using `wget` is documented [here](https://rda.ucar.edu/datasets/ds628.0/docs/ds628.0-Japanese.55-year.Reanalysis.Daily.3-Hourly.and.6-Hourly.Data-wget.download.how-to.txt). Note that this is for 3 and 6-hourly data.