# Tangos Setup Guide

This guide provides information on setting up simulation files run with ChaNGa, using Tangos and Pynbody.

## File Structure

Tangos expects files in a specific format. Two common structures are:

1. `/sim_folder/sim_snapshot_folder/snapshot_data`
   - Simulation-level parameter files are in the root directory.

2. All files in the root directory without organization by timestep.

## Required Files

At minimum, Tangos needs:

- Snapshot data
- Simulation-level .param file
- Snapshot-level .param file

Ensure there are no duplicate snapshot files.

## Additional Requirements

- At least one halo catalog file (e.g., .amgiga or .AHF files)
- For precomputed properties (like H2), include corresponding files (e.g., .H2 files)
- .iord file for every snapshot (crucial for tracking halo properties across timesteps)

## File Naming Conventions

Tangos looks for specific file names:

- Properties: `sim_name_snapshot_number.yourproperties`
- Amiga: `sim_name_snapshot_number.amiga.stat`
- AHF: `sim_name_snapshot_number.z0.000.AHF_halos`

Avoid extra components in filenames (e.g., `sim_name_snapshot_number.0000.z0.000.AHF_halos` will confuse Tangos).

## Setup Options

Two main approaches:

1. Create a custom Tangos handler (advanced)
2. Create a copy of your simulation folder using symlinks (recommended)

Note: Pynbody/Tangos cannot handle symlinked folders.

## Best Practices

- Ensure no unexpected folders contain snapshot data
- For root directory setups, remove unwanted files from your folder of symlinks
- Avoid duplicate snapshots (e.g., AHF data at different overdensities)

## Creating Tangos Database

1. Add your simulation:
   ```
   tangos add sim_folder --backend multiprocessing-N
   ```

2. Create links:
   ```
   tangos link --sim sim_folder --backend multiprocessing-N
   ```
   
   N is the number of threads. For linking, use as many cores as possible, considering RAM limitations (approx. 8GB/core).

## Performance Considerations

- For adding simulations, 10 threads usually suffice
- For linking, maximize core usage based on available RAM
- With 16GB RAM, recommend keeping N around 4, even with more cores available
