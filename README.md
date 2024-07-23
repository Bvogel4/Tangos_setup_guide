# Tangos_setup_guide
This is the information that I have learned to setup simulation files run with changa, using tangos and pynbody

tangos expects files in a certain file format,

I have worked with two examples. 
one looked like /sim_folder/sim_snapshot_folder/snapshot_data
where sim level param files are in the root dir

I have also worked with files where everything is dumped into the root dir, no organization by timestep. 


what files does tangos need?

the most basic files that tangos needs is the snapshot data, as well as a sim level and snapshot .param files. make sure there are no duplicate snapshot files. for example I am working with data that has ahf run at 200 overdensity rather than 178, where the 200 data was organized in another subfolder in a snapshot folder. This causes tangos to add multiple snapshots for each timestep, and tangos cannot find the .param file for the ahf_200 files since it is 2 levels higher and by default it does not look there. 

there are 2 ways then to tell tangos what data you want to use, create a custom tangos handler, that tells tangos what files to look for and where the .param file is (hard) or create a copy of your simulution folder using symlinks(reccomended)
note that pynbody/tangos currently cannot handle symlinked folders, as it will try to load it as a file and cause an error. 

now what files does tangos need specifically to be able to run effective analysis?
it needs at least one halo catalog files, such as .amgiga files or .AHF files.

for any precomputed properties like H2, you will also need to include .H2 files. 

for best resutls make sure there are no unexpected folders that contains any kind of snaphot data at all, it might even be best to make sure the only folders are snapshot files. if everything is dumped in the root files, make sure any files you don't want processes do not exist in your folder of symlinks 

tangos is also looking for specific files names for catalogs and properties. 
propoerties are pretty simple with sim_name_snapshot_number.yourproperties
amiga likewise sim_name_snapshot_number.amiga.stat
but ahf files are a little tricker example
sim_name_snapshot_number.z0.000.AHF_halos

in order for tangos to see your files properly make sure there are not extra components in the name, for example 
sim_name_snapshot_number.0000.z0.000.AHF_halos will confuse tangos 

it is possible to get around the files names by modifying tangos input handlers, but in my experience this is much more difficult. 


#very important
If you are interested in tracking halo properties across timesteps you will need to make sure that there is a .iord file for every snapshot. This is required for pynbody to be able to link halos across snapshots. 


#now that you have prepared your simulation files, how to construct tangos database efficiently?

first add your sim using tangos add sim_folder
recommened options --backend multiprocessing-N
where N is the number of threads you want to use, for the add function, it runs fairly fast at 10 
tangos add sim_folder--backend multiprocessing-10

and likewise to create links 
tangos link --sim sim_folder --backend multiprocessing-N

links takes a while, so you may want to run this with as many cores as you can if you have sufficent ram, in my limited testing around 8GB/core gives sufficent performancces, if you have 16GB of ram I would reccomend to keep N around 4, even if you have more cores

