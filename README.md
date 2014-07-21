SWARM
=====

LUA programs for the Minecraft modification ComputerCraft - shared pathfinding for swarm-influenced semi-autonomous turtles and accompanying tools.

    pastebin run Eaj2GwNK

##What... ... why... ?

This started off as a mining-script for multiple drones, quite a while ago. Since then it has
been rewritten numerous times and evolved into a bunch of programs. It still is basically a 
very long mining script.

The main use for this ingame is to gather large amounts of resources from far away places,
on servers that are constantly running. 
It enables you to control a "swarm" of turtles from a remote place by defining jobs and 
sending them to a server. Drones (turtles) ask the jobserver for work, and will process
jobs one after another. 

You could, for example, send jobs to have a bunch of whole chunks somewhere far away from 
your base mined. 

---

###Required parts:
<ul>
<li>drone - for turtles, executes jobs, be that mining or other stuff turtles like to do.</li>
<li>mapserver - this acts as the shared memory of the drones.</li>
<li>jobserver - this distributes jobs to turtles. send jobs to it with the "job" command.</li>
<li>tools - tools to send jobs, meant to be used on pocketcomputers</li>
</ul>

###Optional parts:
<ul>
<li>infoserver - receives drone communication and shows current stati, also sends data as JSON to URL.</li>
<li>more tools - tools to restart/shutdown all systems, query information ...</li>
</ul>

---

###Noteable system features:
<ul >
<li>Good pathfinding based on A* with a bunch of optimizations, caching, and shared map. Works well with exploration as well known areas. (Considering turtles are basically blind.)</li>
<li>Mining-functions included in API, like digArea or digChunk (100% quarry pattern, refueling, unloading...)</li>
<li>Drones can specify slots for blocks not be mined, enabling fast mining of all ore while leaving ~60% unwanted untouched and dropping the other.</li>
<li>Binary mapfile format for mapserver, so loading/saving map is faster and takes less space.</li>
<li>Drones are aware of each other, can be individually configured, or have their config overwritten by a job.</li>
<li>More, probably..</li>
</ul>

---

###Known problems:
<ul>
<li>Too many drones will makes everything CC stop working, seems to depend on server-power though.</li>
<li>Still a bit tricky to set up, with the ids and stuff.</li>
<li>Lots of bugs!</li>
</ul>

---

##How?
###Setup:
You will need:
<ul>
<li>ComputerCraft</li>
<li>http api enable in CC configs (only needed for installer and infoserver)</li>
<li>increase the range of wireless modems (not needed, but no fun without)</li>
</ul>

You will need ingame:
<ul>
<li>A Working GPS.</li>
<li>At least one turtle.</li>
<li>At least 3 computers or 2 computers and 1 pocketcomputer.</li>
<li>Wireless modems for all systems.</li>
<li>Some fuel to get the drones started.</li>
</ul>

To use the installer (recommended) type
    pastebin run Eaj2GwNK
on every system, and follow instuctions on screen.

You will need to change configs by hand, default configs should be generated
after install.

Positions are written like this depending on target function:
<ul>
<li>{x,y,z}</li>
<li>{x,y,z,direction}</li>
<li>{xMin,yMin,zMin,xMax,yMax,zMax}</li>
<li>{xMin,yMin,zMin,xMax,yMax,zMax,direction}</li>
</ul>

Directions are interpreted like this:
<ul>
<li>0 North</li>
<li>1 East</li>
<li>2 South</li>
<li>3 West</li>
<li>4 Up</li>
<li>5 Down</li>
</ul>

So a drone that should have its home at 1,2,3 and unloads items to a chest 
below that spot and refuels from a chest above it would have the following 
config entries (among others):
```lua
    -- it does not matter in this case, but it shall face north.
    config["homePos"] = {1,2,3,0}
    config["rechargePos"] = {1,2,3,4}
    config["storagePos"] = {1,2,3,5}
```

###Examples:
Jobs can be send by specifying a file or a string containing a valid job.<br/>
```lua
  -- Type those on a computer with wireless modem after setup:
  
    -- This prints Hello World! on the screen of the drone that receives the job.
  job "print('Hello World!')"
  
  -- This will make a drone move to the location 1,2,3 facing north, it will keep trying indefinitely.
  job "moveTo({1,2,3,0},-1,0)"

  -- This will make a drone go to chunk 1,2 and mine it from z=120 on downwards.
  -- The drone's config defines what gets mined, if nothing is specified, everything is mined.
  job "digChunk(1,2,120,0)"
  
  -- This will do the same as above, but overwrites the drone's saved config to have it not mine blocks
  -- like the ones it has in it's inventory slots 1 and 2.
  job "config["ignoreSlots"] = {1,2} digChunk(1,2,120,0)"
  
  -- Just like above, but this time also drop blocks matching ignoreSlots that drone had to mine in order
  -- to dig its path. Will always keep 1 items in ignoreSlots.
  job "config["dropIgnoreSlots"] = true config["ignoreSlots"] = {1,2} digChunk(1,2,120,0)"
```

---

You have to take care of chunkloading yourself. But, alas, take a look at this: <a href="http://www.computercraft.info/forums2/index.php?/topic/18156-mc-16-cc-158-163-turtle-chunkloaders-mining-chunkloaders-crmod/">CR's turtle peripherals</a>
