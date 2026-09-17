# Guide to MITgcm Output variables

| **Variable** | **Description** | **Units** |
| --- | --- | --- |
| RC | R coordinate of cell centre | m |
| RF | R coordinate of cell interface | m  |
| RU | R coordinate of upper interface | m |
| RL  | R coordinate of lower interface | m  |
| drC | r cell centre separation  |  |
| drF | r cell face separation |  |
| XC | X coordinate of cell centre  | degree_east |
| YC | Y coordinate of cell centre  | degree_north |
| XG | X coordinate of cell corner  | degree_east |
| YG | Y coordinate of cell corner  | degree_north |
| dxC | x cell centre separation  |  |
| dxF | x cell face separation  |  |
| dyF | y cell face separation |  |
| dxG | x cell corner separation |  |
| dyG | y cell corner separation |  |
| dxV | x v-velocity separation |  |
| dyU | y u-velocity separation |  |
| rA | r-face area at cell corner |  |
| rAw | r-face area at U point  |  |
| rAs | r-face area at V point |  |
| rAz | r-face area at cell corner  |  |
| fCori | Coriolis (f) at cell centre  |  |
| fCoriG | Coriolis (f) at cel corner  |  |
| HFacC | Fraction of a vertical grid cell that contains fluid (at the cell centre) |  |
| HFacW | Fraction of a vertical grid cell that contains fluid (at the west face) |  |
| HFacS | Fraction of a vertical grid cell that contains fluid (at the south face) |  |
| RAC | area of cell centre (1/diff(XC)/diff(YC)) |  |
| RAS | area of southern point (1/diff(XC)/diff(YG) |  |
| RAW | area of western point (1/diff(XG)/diff(YC) |  |
| RAZ | area of vorticity (zeta) point (1/diff(XG)/diff(YG)) |  |