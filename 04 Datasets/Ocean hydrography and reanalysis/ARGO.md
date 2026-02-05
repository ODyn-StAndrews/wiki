# ARGO

Snapshots of ARGO data can be downloaded [here](https://www.seanoe.org/data/00311/42182/). This site regularly updates with the latest automatically QC'd snapshop (roughly monthly).

For the global dataset, scroll down to the data access section, right click and copy the link address for the `Global GDAC Argo data files (2026-01-08 snapshot)`, where the date corresponds to the particular snapshot of interest. You can use `wget` to retrieve that file, e.g.
```
wget https://www.seanoe.org/data/00311/42182/data/125185.tar.gz
```
where the URL is the copied link.

This approach downloads a zipped `tar`-ball. You need to first unzip that with `gunzip`. The remaining `tar`-ball contains zipped data at a variety of levels, including both single files with all profiles and individual files with each individual profile. This makes unpacking a heavy lift. Instead, you can extract only the `*_prof.nc` files using
```
tar -I pigz --wildcards -xvf 125185.tar.gz '*prof.nc'
```
Note that the `-I pigz` is there to run the unzip procedure in parallel. It is not always available on all machines.