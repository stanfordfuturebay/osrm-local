# Install and Run Notes

These instructions are for creating a local OSRM (Open Source Routing Machine) instance on your machine, which will enable much more computationally-expensive uses of functions in osrm. For simpler tasks you don't need to do this, as the functions will use the demo cloud server. But if you are doing A LOT of routing, then this is a great investment of time to make. Reach out to us if you notice any errors or need any help.

## Windows (Ubuntu)

### Install Ubuntu

We've prepared some scripts that need to be run in Linux on the Ubuntu Operating System, so you'll need to install Ubuntu on Windows to execute them. It's free and the easiest way to get it is through the [Microsoft Store](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6). You'll need to create an admin password for Ubuntu during the install process. This password is separate from your Windows password. 

### Installing

The first few lines need to be run to build the back-end infrastructure of OSRM. 

You might be familiar with Ctrl+C and Ctrl+V but find that it doesn't work in the Ubuntu terminal -- you can instead use right-click to paste, and selecting within the terminal serves as copy. So you can Ctrl+C one line at a time from below and right-click paste into the terminal. This course isn't meant to get into Linux any deeper, but if you've never used it before, two other bare minimum commands to know:

- ```cd```: change directory. You start in your home directory, and once you have other folders to travel to you essentially just type ```cd [name of the folder]```. You see that done in the code below. You can go back up to a parent folder using ```cd ..```. To go all the way back to your home, use ```cd ~```. A few more examples [here](https://www.tecmint.com/cd-command-in-linux/).
- ```ls```: list. Lists what is in the current directory. Just type it whenever. More examples [here](https://www.tecmint.com/15-basic-ls-command-examples-in-linux/).

Other notes: 

- You will need to create a Github account, as you will be prompted for your username and password.
- You will be prompted for your Ubuntu password. 

Run the following lines one-by-one. This is based on the methodology found [here](https://datawookie.netlify.com/blog/2017/09/building-a-local-osrm-instance/).

```bash
sudo apt update
sudo apt install -y git cmake build-essential jq htop
sudo apt install -y liblua5.2-dev libboost-all-dev libprotobuf-dev libtbb-dev libstxxl-dev libbz2-dev

git clone --branch v5.18.0 https://github.com/Project-OSRM/osrm-backend.git

cd osrm-backend/

mkdir build
cd build/
cmake ..

make

sudo make install
```

### Downloading OSM Data

Create a map directory and change to it to prevent clutter in the parent directory.  Run the ```wget``` function inside the map directory. 

There are multiple ways to grab OSM data, depending on what you need for your analysis.

1. Go to the [OpenStreetMap Export page](https://www.openstreetmap.org/export#map=7/36.748/-120.256), select "Manually select a different area", and then crop a region that contains all of the the transportation network you need. In most cases the best option is to right-click on "Overpass API", copy the link address, and then use that to replace the URL in the ```wget``` line below. The default URL shown below grabs a section of map that contains SF, the Peninsula, and Silicon Valley.
2. It's possible your selection is too big for the Overpass API (e.g. bigger than the Bay Area), in which case you likely need to go to [Geofabrik](https://download.geofabrik.de/) and download all of CA. Your call then would look like ```wget -O ca.osm.pbf "https://download.geofabrik.de/north-america/us/california-latest.osm.pbf"```. Note the file type has changed to osm.pbf.

```
cd ~/osrm-backend
mkdir map
cd map

wget -O mymap.osm "https://overpass-api.de/api/map?bbox=-122.5298,37.2101,-121.7525,37.8136"
```

### Creating the OSRM Object and Running

Run the lines below while still in the map directory. It creates all of the needed graph objects inside the directory that the input OSM file is located in. By doing this, it prevents the parent directory from getting too cluttered and makes deletion of files easy to create another graph object. Although, if you just use a different name then it's not a problem. 

Imperfect explanation of what's going on:

- 1st line: creating an OSRM object from the data downloaded, using the car profile (this can be switched with bicycle.lua and foot.lua)
- 2nd line: creating the contraction hierarchies, which facilitate finding the shortest route between two points.
- 3rd line: runs OSRM. Note, only this line has to be run in the future to get OSRM up and running. Have to be inside of the map directory to run this line as is -- otherwise modify extensions as appropriate. 

```
../build/osrm-extract mymap.osm -p ../profiles/car.lua
../build/osrm-contract mymap.osrm
../build/osrm-routed mymap.osrm
```

Once the last line above has been run, you'll see it say ```running and waiting for requests```. then keep the Ubuntu window open and then switch to R. In your scripts, make sure you have the following at the top:

```
library(osrm)
options(osrm.server = "http://127.0.0.1:5000/")
```

After all of this has been done once, then in the future, to get your local server, just do the following:

1. Open the Ubuntu app
2. Type the following commands

```
cd osrm-backend
cd map
../build/osrm-routed mymap.osrm
```

## MacOS

For help with the Mac version, reach out to Max, who is continuing to streamline the process.

You'll need to start by installing homebrew. You can do that by running the command below.
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

### Installing

Followed roughly the first half of the instructions [here](https://gist.github.com/jyt109/76eba9b502e2c90bb728) to get it running. When trying to follow the quickstart guide [here](<https://github.com/Project-OSRM/osrm-backend/wiki/Building-OSRM>), it seems like the "mason" folder isn't created. But, once I install all of the dependencies etc., I then run the 

```
cmake ../ -DENABLE_MASON=1
make
```

lines and it seems to have worked. In fact, I know it works because when I go to start it up following the instructions [here](<https://github.com/Project-OSRM/osrm-backend/wiki/Running-OSRM>), it does in fact work. The final code I ran should look something like this: 

#### Install example code

Note that during this process, you may get a number of error messages if any of the packages are already on your machine. No need to worry as this is okay. 

```
# from the gist
brew install boost git cmake protobuf libzip libstxxl libxml2 osm-pbf lua51 luajit luabind tbb
brew uninstall lua 
brew install GDAL                                          
brew install osmosis
brew install wget
git clone https://github.com/Project-OSRM/osrm-backend.git 
cd osrm-backend                                            
mkdir -p build                                             
cd build                                                   

# from the osrm wiki
cmake ../ -DENABLE_MASON=1
make

```

### Running

I first tried running these commands from the [wiki](<https://github.com/Project-OSRM/osrm-backend/wiki/Running-OSRM>).

Simple checklist that will make sense after reading through the code and comments. 

* Double check that the file names are correct. 
* You actually have to use "./" before the commands to get them to run AND if things are floating around in different directories, you have to include that as well. 
  * May not actually need the "./" but it doesn't hurt. 

#### From the wiki

```
# NorCal extract. 
wget https://download.geofabrik.de/north-america/us/california/norcal-latest.osm.pbf

# The following functions extract the compressed file and get it read to run. Note that this is it appears on the website. 
osrm-extract norcal-latest.osm.pbf

# On online post and the CH, the alternate version of the first line is as follows, this is the one you should probably use. When attempting to run the first version as is, you may get a message that says something like "car.lua" doesn't exist. 

osrm-extract norcal-latest.osm.pbf -p profiles/car.lua
osrm-partition norcal-latest.osm
osrm-customize norcal-latest.osm

osrm-routed --algorithm=MLD norcal-latest.osm.osrm 
```

#### Actual commands

```
# This is the terminal input with the current directory set to the osrm-backend file directly downloaded from GitHub

./build/osrm-extract norcal-latest.osm.pbf -p profiles/car.lua
build/osrm-partition norcal-latest.osm
build/osrm-customize norcal-latest.osm
build/osrm-routed --algorithm=MLD norcal-latest.osm
```
