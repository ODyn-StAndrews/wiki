# BSOSE

[Homepage](http://sose.ucsd.edu/)  
Download location: As for homepage

*B-SOSE may be migrated to [ERDDAP](https://equator.ucsd.edu/erddap/index.html) in the near future.*

**Download procedure**
* Various variable files are available for different iterations at the above address.
* Click to download or issue `wget` command.
* For `wget` retrieval, you can get all of the files on a given page by copying the URLs for the files into a `.csv` file, then issuing `wget -i filelist.csv`
    * One straightforward way to construct a `.csv` file with all the URLs is to click on the iteration website (taking you, for example for iteration 136, [here](http://sose.ucsd.edu/SO6/ITER136/)).
    * Copy all these file names, then paste them into a file using `vim`, and save with `.csv` ending.
    * In `vim` add the URL address to every line (`ctrl+v` to select block > select the start of all lines > `shift+i` then enter the URL precursor > `esc`).
    * Attach `-nc` flag to avoid repeating downloads that already exist.

* For [ITER133](http://sose.ucsd.edu/BSOSE6_iter133_solution.html), there is a a full filelist [here](https://hackmd.io/oBZk3T2AQiqt92CqCjyLkQ).
* *Note that standard approaches to use `wget` with wildcards, e.g. [here](https://unix.stackexchange.com/questions/117988/wget-with-wildcards-in-http-downloads) or [here](https://stackoverflow.com/questions/18107236/using-wildcards-in-wget-or-curl-query), did not work for me for these data.*