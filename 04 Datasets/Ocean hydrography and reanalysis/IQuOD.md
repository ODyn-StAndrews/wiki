# IQuOD

International Quality-controlled Ocean Database

[Homepage](https://www.iquod.org/index.html)

# Data access
[v0.1 (NCEI)](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.nodc:0170893)

## Download procedure
IQuOD data are available through a few channels. A simple option is to download from the `https` server using `wget`. Executing a simple `bash` script such as the following will download all data into your local folder (wherever the script is executed) with the same folder as on the server:
```
BASE="https://www.ncei.noaa.gov/data/oceans/ncei/archive/data/0170893/"

wget -r -np -nH --cut-dirs=6 \
     --reject "index.html*" \
     --timestamping --continue \
     --wait=1 --random-wait \
     -e robots=off \
     "$BASE"
```
