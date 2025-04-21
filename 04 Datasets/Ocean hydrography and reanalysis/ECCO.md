# ECCO

[Homepage](https://www.ecco-group.org/home.cgi)  
Download location:
* [ECCO Drive](https://ecco.jpl.nasa.gov/drive/) *Requires EarthData account*
* [Mirror site](https://podaac.jpl.nasa.gov/ECCO) when Drive is down

**Download procedure**
* ECCO Drive
    * Instructions for using `wget` to download from ECCO Drive at the command line are available [here](https://www.ecco-group.org/docs/wget_download_multiple_files_and_directories.pdf).
    * You need your "WebDAV/Programmatic API" credentials for this, which can be found by navigating to the  ECCO Drive site above.
    * Basic `wget` command (to get all of ECCOV4r4):
    * ```wget --user=ifenty --password=ABCD -r -nc -np -nH --cut-dirs=2 https://ecco.jpl.nasa.gov/drive/files/Version4/Release4/```