# Install and Run Notes

These instructions are for creating a local OSRM (Open Source Routing Machine) instance on your machine, which will enable much more computationally-expensive uses of functions in osrm. For simpler tasks you don't need to do this, as the functions will use the demo cloud server. But if you are doing A LOT of routing, then this is a great investment of time to make. Reach out to us if you notice any errors or need any help.

## Windows (Ubuntu)

### Install Ubuntu

We've prepared some scripts that need to be run in Linux on the Ubuntu Operating System, so you'll need to install Ubuntu on Windows to execute them. It's free and the easiest way to get it is through the [Microsoft Store](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6). You'll need to create an admin password for Ubuntu during the install process. This password is separate from your Windows password. 

### Installing

The following commands clone a Github repo with scripts then run them. You might be familiar with Ctrl+C and Ctrl+V but find that it doesn't work in the Ubuntu terminal -- you can instead use right-click to paste, and selecting within the terminal serves as copy. So you can Ctrl+C one line at a time from below and right-click paste into the terminal. This course isn't meant to get into Linux any deeper, but if you've never used it before, two other bare minimum commands to know:

- ```cd```: change directory. You start in your home directory, and once you have other folders to travel to you essentially just type ```cd [name of the folder]```. You see that done in the code below. You can go back up to a parent folder using ```cd ..```. To go all the way back to your home, use ```cd ~```. A few more examples [here](https://www.tecmint.com/cd-command-in-linux/).
- ```ls```: list. Lists what is in the current directory. Just type it whenever. More examples [here](https://www.tecmint.com/15-basic-ls-command-examples-in-linux/).

The ```chmod``` commands set the permissions. You will be prompted for your ubuntu password. 

```
git clone https://github.com/stanfordfuturebay/osrm_local
cd osrm_local
chmod 755 install_script
./install_script

chmod 755 first_run_script
./first_run_script

chmod 755 run_script
./run_script
```

Continue to copy and paste each line below to build the local OSRM instance. This is based on the methodology found [here](https://datawookie.netlify.com/blog/2017/09/building-a-local-osrm-instance/).

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

1. Go to the [OpenStreetMap Export page](https://www.openstreetmap.org/export#map=7/36.748/-120.256), select "Manually select a different area", and then crop a region that contains all of the the transportation network you need. In most cases the best option is to right-click on "Overpass API", copy the link address, and then use that to replace the URL in the ```wget``` line below. 

```
mkdir map
cd map

wget -O test.osm "https://overpass-api.de/api/map?bbox=-121.5802,37.7419,-120.9698,38.1437"
```

### Creating the OSRM Object and Running

Run the lines below while still in the map directory. It creates all of the needed graph objects inside the directory that the input OSM file is located. By doing this, it prevents the parent directory from getting too cluttered and makes deletion of files easy to create another graph object. Although, if you just use a different name then it's not a problem. 

Imperfect explanation of what's goin on:

1st line:  creating an OSRM object from the data downloaded

2nd line: creating the contraction hierarchies (see the documentation on GitHub for explanation)

3rd line: runs OSRM. Note, only this line has to be run in the future to get OSRM up and running. Have to be inside of the map directory to run this line as is, otherwise modify extensions as appropriate. 

```
../build/osrm-extract test.osm -p ../profiles/car.lua
../build/osrm-contract test.osrm
../build/osrm-routed test.osrm
```

#### Future things to consider

* 


## MacOS


### Installing

So, somehow I got it installed and I’m not really quite sure how that happened. 

Followed roughly the first half of the instructions [here](https://gist.github.com/jyt109/76eba9b502e2c90bb728) to get it running. When trying to follow the quickstart guide [here](<https://github.com/Project-OSRM/osrm-backend/wiki/Building-OSRM>), it seems like the "mason" folder isn't created. But, once I install all of the dependencies etc., I then run the 

```
cmake ../ -DENABLE_MASON=1
make
```

lines and it seems to have worked. In fact, I know it works because when I go to start it up following the instructions [here](<https://github.com/Project-OSRM/osrm-backend/wiki/Running-OSRM>), it does in fact work. The final code I ran should look something like this: 



#### Install example code

```
# from the gist
brew install boost git cmake protobuf libzip libstxxl libxml2 osm-pbf lua51 luajit luabind tbb
brew uninstall lua 
brew install GDAL                                          
brew install osmosis                                       
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

Simple checklist that will make sense after reading throught the code and comments. 

* Double check that the file names are correct. 
* You actually have to use "./" before the commands to get them to run AND if things are floating around in different directories, you have to include that as well. 
  * May not actually need the "./" but it doesn't hurt. 



#### From the wiki

```
# This just downloads the osm file that everyone seems to use as an example. 
wget http://download.geofabrik.de/europe/germany/berlin-latest.osm.pbf

# The following functions extract the compressed file and get it read to run. Note that this is it appears on the website. 
osrm-extract berlin.osm.pbf

# On online post and the CH, the alternate version of the first line is as follows, this is the one you should probably use. When attempting to run the first version as is, you may get a message that says something like "car.lua" doesn't exist. 

osrm-extract berlin-latest.osm.pbf -p profiles/car.lua
osrm-partition berlin.osrm
osrm-customize berlin.osrm

osrm-routed --algorithm=MLD berlin.osrm 
```



#### Actual commands

```
# This is the terminal input with the current directory set to the osrm-backend file directly downloaded from GitHub

./build/osrm-extract berlin-latest.osm.pbf -p profiles/car.lua
build/osrm-partition berlin-latest.osrm
build/osrm-customize berlin-latest.osrm
build/osrm-routed --algorithm=MLD berlin-latest.osrm
```
