# AccessViz

A Python toolkit for managing, joining, visualizing, and comparing travel time data from the [Helsinki Region Travel Time Matrix](https://blogs.helsinki.fi/accessibility/helsinki-region-travel-time-matrix/) dataset.

AccessViz has four main components:

1. **FileFinder** — locates travel time matrix files on disk based on a list of YKR ID values
2. **TableJoiner** — joins matrix files to the `MetropAccess_YKR_grid` shapefile, producing spatial layers
3. **Visualizer** — creates static (matplotlib) or interactive (folium) maps of travel times by mode
4. **Comparison tool** — compares travel times or distances between two travel modes

## Getting the data

This repository does **not** include the Travel Time Matrix data itself (13,000+ files, several GB) — it's excluded via `.gitignore`.

1. Download the Travel Time Matrix dataset (2013 / 2015 / 2018) and the `MetropAccess_YKR_grid.shp` shapefile from the https://blogs.helsinki.fi/accessibility/helsinki-region-travel-time-matrix/
2. Extract the data into a folder named `data/` in the root of this project, so the structure looks like:

```
AccessViz/
├── data/
│   ├── HelsinkiRegion_TravelTimeMatrix2013/
│   │   ├── 5785xxx/
│   │   │   ├── travel_times_to_5785640.txt
│   │   │   └── ...
│   │   └── ...
│   └── MetropAccess_YKR_grid/
│       ├── MetropAccess_YKR_grid.shp
│       └── ...
├── output/      (created automatically — joined/compared spatial files)
├── images/      (created automatically — static and interactive maps)
└── AccessViz.ipynb
```

All paths in the notebook are **relative**, so as long as `data/` sits alongside the notebook, no path edits should be needed.

## Requirements

- Python 3.x with `geopandas`, `pandas`, `matplotlib`, `folium`, and `contextily`
- A conda environment (e.g. the course's `autogis` environment) is recommended

## Usage

Each component is a standalone function and can be used independently, or chained together as shown in the driver cell at the bottom of `AccessViz.ipynb`:

```python
# 1. Find files for a list of YKR IDs
files = filefinder(id_list, Data_Directory)

# 2. Join matrix files to the grid, save as .shp or .gpkg
out_files = tablejoiner(files, grid, Output_Directory, output_type)

# 3. Visualize travel times (static or interactive)
visualiser(out_files, columns, travel_mode, map_type, Output_Image_Directory)

# 4. Compare two travel modes
comparison(out_files, modes, comp_type, Comp_Output_Directory)
```

## Design notes

- **Independent functions.** Each function takes filepaths (not in-memory objects) as input and re-reads from disk where needed, so any component can be run on its own, in any order, using previously saved output from another session — not just chained in one continuous run.
- **NoData handling.** The dataset marks missing values as `-1`. This is preserved as-is through FileFinder and TableJoiner rather than converted to `NaN`, so each downstream function can decide how to treat it: Visualizer filters `-1` out before plotting; the Comparison tool explicitly forces any result row where either input was `-1` back to `-1`, per the assignment spec.
- **Travel mode defaults.** Each travel mode (car, public transport, walking) maps to one specific time/distance column (see `columns` / `time_columns` / `distance_columns` dictionaries in the notebook). A midday time slice was chosen as the default for each mode; column choices are documented inline and can be edited for a different time-of-day or dataset year.
- **Duplicate/missing ID handling.** FileFinder warns and skips duplicate IDs and IDs with no matching file, rather than failing the whole batch.
- **Classification scheme.** Static maps use a `quantiles` classification scheme via `mapclassify`, chosen for producing evenly populated, readable class breaks across the skewed travel-time distribution"].

## Known limitations

- Column names are based on the Helsinki Region Travel Time Matrix 2013 dataset; using the 2015/2018 datasets may require updating the mode-to-column dictionaries if column names differ.
- Static and interactive maps may use slightly different default binning methods (matplotlib/mapclassify `quantiles` vs. folium's default `Choropleth` binning); this is a known inconsistency, not a bug.

## Author

Rahul Madhukumar — Geo-Python and Automating GIS Processes (‘AutoGIS’) developed by the Department of Geosciences and Geography at the University of Helsinki, Finland. 
