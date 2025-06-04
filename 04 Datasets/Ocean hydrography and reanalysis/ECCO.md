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

**Troubleshooting**
* `ERROR: The certificate of ‘ecco.jpl.nasa.gov’ is not trusted. ERROR: The certificate of ‘ecco.jpl.nasa.gov’ doesn't have a known issuer.`
   * Add the flag `--no-check-certificate' after your password flag.
   * NOTE: Only use this flag if you are not worried about security and checking the validity of the website's certificate. Seeing as we're working with the official ECCO website, it is acceptable in this case but use with caution in general.
