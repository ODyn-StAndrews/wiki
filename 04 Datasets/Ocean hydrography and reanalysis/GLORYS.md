# GLORYS

[Homepage](https://data.marine.copernicus.eu/product/GLOBAL_MULTIYEAR_PHY_001_030/description)
Download location: [Copernicus Marine Data Store](https://data.marine.copernicus.eu/product/GLOBAL_MULTIYEAR_PHY_001_030/description)

**Download procedure**:
*Copernicus Marine Toolbox*
* Downloads of Copernicus Marine Data Store are handled through the [Copernicus Marine Toolbox](https://help.marine.copernicus.eu/en/articles/7949409-copernicus-marine-toolbox-introduction). This is a python-oriented toolbox for accessing, subsetting, and downloading CMS data, either through the Command Line Interface or the Python API.
* Install the toolbox in your local environment, following instructions [here](https://help.marine.copernicus.eu/en/articles/7970514-copernicus-marine-toolbox-installation).
* Follow the instructions on [the main page](https://help.marine.copernicus.eu/en/articles/7949409-copernicus-marine-toolbox-introduction) to see the various uses of the toolbox.
*Defining a GLORYS subset*
- To determine the commands with which to download GLORYS data specifically, navigate to the download location above.
- Navigate to the "Data Access Tab"
* For the relevant dataset (e.g. "Daily"), click on "Form" button under "Subset" category to go to the data selection GUI (Copernicus sign-in required).
* Select the required variables, region, and date range.
* If the dataset is small enough and you want to download *locally* (i.e. to the computer in which you have launched the browser) you will be able to do so using the clickable icon in the upper right.
* If the dataset is too large, or if you're downloading directly to a remote machine, click on the "Automate" button on the upper right. This will open a pop up with both a "Command-Line Interface" and a "Python API" tab, with the exact commands that you need to use to download the data. Copy and paste these into a python script (locally if you want to download to your local computer, or on a remote machine), and run that script.

An example of the python approach to downloading daily GLORYS data for the North Atlantic is given below:

```
import copernicusmarine

copernicusmarine.subset(
  dataset_id="cmems_mod_glo_phy_my_0.083deg_P1D-m",
  variables=["mlotst"],
  minimum_longitude=-100,
  maximum_longitude=20,
  minimum_latitude=0,
  maximum_latitude=80,
  start_datetime="1993-01-01T00:00:00",
  end_datetime="2021-06-30T00:00:00",
  minimum_depth=0.49402499198913574,
  maximum_depth=0.49402499198913574,
  file_format='zarr'
)
```

By default, the tool downloads `.nc` files, but these can get large and unwieldy. For large datasets, downloading as `zarr` store helps with data IO. Other useful options for the `subset` command are to issue your Copernicus `username` and `password` directly, and to specify an `output_directory`.
