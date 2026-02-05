# ARGO

Snapshots of ARGO data can be downloaded [here](https://www.seanoe.org/data/00311/42182/). This site regularly updates with the latest automatically QC'd snapshop (roughly monthly).

For the global dataset, scroll down to the data access section, right click and copy the link address for the `Global GDAC Argo data files (2026-01-08 snapshot)`, where the date corresponds to the particular snapshot of interest. You can use `wget` to retrieve that file, e.g.
```
wget https://www.seanoe.org/data/00311/42182/data/125185.tar.gz
```
where the URL is the copied link.

This approach downloads a zipped `tar`-ball. Extracting relevant data from this `tar`-ball can be rather a heavy lift. You can see what's inside the `tar`-ball with
```
tar -tzf file.tar.gz
```
or 
```
tar -tzf file.tar.gz | sed 's|[^/]*/| |g'
```
to make it like a directory tree.

One option is to extract only the core argo files and then only the full profile files (rather than each profile individually). Some bash code for doing that is:
```
set -euo pipefail

MAIN="125185.tar.gz"
WORK="./_work_125185"
OUT="./profiles"

mkdir -p "$WORK" "$OUT"

echo "=== Stage 1: Extracting *_core.tar.gz from $MAIN ==="
tar -xzvf "$MAIN" -C "$WORK" --wildcards --no-anchored '*_core.tar.gz'

echo "=== Stage 2: Extracting *prof.nc from each core archive ==="

find "$WORK" -type f -name "*_core.tar.gz" -print0 | while IFS= read -r -d '' core; do
  echo "---- Processing $core ----"
  tar -xzvf "$core" -C "$OUT" --wildcards --no-anchored '*prof.nc'
done

echo "=== Done ==="
```