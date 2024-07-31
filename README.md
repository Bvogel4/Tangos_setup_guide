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
1. First set envirment variables
   ```
   export TANGOS_DB_CONNECTION=/path/to/data.db
   export TANGOS_SIMULATION_FOLDER=/path/to/simfolder
   ```

3. Add your simulation:
   ```
   tangos add sim_folder --backend multiprocessing-N
   ```

4. Create links:
   ```
   tangos link --sim sim_folder --backend multiprocessing-N
   ```
   
   N is the number of threads. For linking, use as many cores as possible, considering RAM limitations (approx. 8GB/core).

## Performance Considerations

- For adding simulations, 10 threads usually suffice
- For linking, maximize core usage based on available RAM
- With 16GB RAM, recommend keeping N around 4, even with more cores available

## Custom properties
There will come a time when you want to write properties into you tangos database that tangos does not have be default. 
create a python file myproperty.py and add it to tangos and your python path
```
export PYTHONPATH=/path/of/myproperties #not the file itself, but its folder
export TANGOS_PROPERTY_MODULES=mytangosproperty
```
while in python
```python
import myproperties
```

#myproperties.py
what should my properties look like? 

I work a lot with shapes, so here is what that looks like
```python
from tangos.properties.pynbody import PynbodyPropertyCalculation
from tangos.properties.pynbody.centring import centred_calculation
from pynbody.analysis.halo import shape
class Shape(PynbodyPropertyCalculation):
    """
    Calculate the axis ratio of the stellar and dark matter components of a halo, only for where stars and dark matter both exist.
    """
    names = ['ba_s', 'ba_d', 'ca_s', 'ca_d', 'rbins']
    def __init__(self, simulation):
        super().__init__(simulation)

    def requires_property(self):
        return ["shrink_center"]


    def _get_shape(self, halo):

        nbins = 100
        #get stellar shapes
        stars = halo.s
        rbins, axis_s,D_in_bin,rot_s = shape(stars,nbins=nbins,rmin=None,rmax=None,ndim=3,bins='equal',max_iterations=20,tol=1e-3)
        self.rbins = rbins
        #get dark matter shapes
        rmin = rbins[0]
        rmax = rbins[-1]
        dm = halo.dm
        r_d, axis_d,D_in_bin,rot_d = shape(dm,nbins=nbins,rmin=rmin,rmax=rmax,ndim=3,bins='equal',max_iterations=20,tol=1e-3)

        a_s = axis_s[:,0]
        b_s = axis_s[:,1]
        c_s = axis_s[:,2]
        ba_s = b_s/a_s
        ca_s = c_s/a_s

        a_d = axis_d[:,0]
        b_d = axis_d[:,1]
        c_d = axis_d[:,2]
        ba_d = b_d/a_d
        ca_d = c_d/a_d

        return ba_s, ba_d, ca_s, ca_d, rbins
    @centred_calculation
    def calculate(self, halo, existing_properties):
        ba_s, ba_d, ca_s, ca_d, rbins = self._get_shape(halo)
        return ba_s, ba_d, ca_s, ca_d , rbins


    def plot_xlog(self):
        return False

    def plot_ylog(self):
        return False
```

example of a write that returns all properties to the database is 
```
tangos write ba_s
```
even though I did not list all the properties.


#live property calculation
working with the sim files itself is slow, so what if you want to have a calculation only on things in the data base.
then use a live property

```python
from tangos.properties import LivePropertyCalculation
class SmoothAxisRatio(LivePropertyCalculation):
    names = ['ba_s_smoothed', 'ba_d_smoothed', 'ca_s_smoothed', 'ca_d_smoothed', 'ba_s_at_reff', 'ba_d_at_reff', 'ca_s_at_reff', 'ca_d_at_reff']

    def requires_property(self):
        return ['ba_s', 'ba_d', 'ca_s', 'ca_d', 'rbins', 'reff']
    @staticmethod
    def smooth_shape(x, y, k=3):
        return UnivariateSpline(x, y, k=k)

    def calculate(self, halo, existing_properties):
        rbins = existing_properties['rbins']
        ba_s_smoothed = self.smooth_shape(rbins, existing_properties['ba_s'])
        ba_d_smoothed = self.smooth_shape(rbins, existing_properties['ba_d'])
        ca_s_smoothed = self.smooth_shape(rbins, existing_properties['ca_s'])
        ca_d_smoothed = self.smooth_shape(rbins, existing_properties['ca_d'])
        reff = existing_properties['reff']
        ba_s_reff = ba_s_smoothed(reff)
        ba_d_reff = ba_d_smoothed(reff)
        ca_s_reff = ca_s_smoothed(reff)
        ca_d_reff = ca_d_smoothed(reff)
        return ba_s_smoothed(rbins), ba_d_smoothed(rbins), ca_s_smoothed(rbins), ca_d_smoothed(rbins), ba_s_reff, ba_d_reff, ca_s_reff, ca_d_reff
```

don't forget to import the neccassary classes eg.
```python

from tangos.properties import LivePropertyCalculation
```

## Writing properties to your database
it may be convieient to to compute a property for only progenitors of a certain halo. 

you can do that like this 
```
tangos write ba_s Mvir reff --sim r431.romulus25.3072g1HsbBH --include-only="latest().halo_number()==2" --include-only="t()>10" --backend multiprocessing-10 ```

