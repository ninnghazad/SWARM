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

Through the mapserver the explored areas are known to all drones, which speeds up exploration
significantly when multiple drones navigate the same area.
The whole map is automatically backed up to disk so it is not lost on restart. 

You don't have to manually place the turtles somewhere or program in any paths, they will 
navigate on their own and find paths between coordinates. 

You could, for example, have the drones mine large areas of whole chunks far away from your base. 
Drones that are free would explore the area to find a path to the mining-zone and dig away, 
bringing back the loot to your base. 


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
<li>Full job queue, jobs wait until a drone is free to process them.</li>
<li>Good pathfinding based on A* with a bunch of optimizations, caching, and shared map. Works well with exploration as well known areas. (Considering turtles are basically blind.)</li>
<li>Efficient mining-functions included in API, like digArea or digChunk (100% quarry pattern, refueling, unloading...)</li>
<li>Drones can specify slots for blocks not be mined, enabling fast mining of all ore while leaving ~60% unwanted untouched and dropping the other.</li>
<li>Binary mapfile format for mapserver, so loading/saving map is faster and takes less space.</li>
<li>Drones are aware of each other, can be individually configured, or have their config overwritten by a job.</li>
<li>More, probably..</li>
</ul>

---

###Known problems and important notes:
<ul>
<li>Too many drones will make everything CC stop working, seems to depend on server-power though.</li>
<li>Still a bit tricky to set up, with the ids and stuff.</li>
<li>Since the jobserver, restarting jobs at same spot after crash needs some work.</li>
<li>Lots of bugs!</li>
<li>Turtles are basically blind, they only sense what they can bump into. That means that for a turtle to navigate a large space, especially if it is a dead end, it has to visit every spot that may be part of a path. And by visit i mean actually move to that spot, it cannot see the obvious exit until it is standing right in it. This has nothing to do with this software, it is just a limitation of blind turtles.</li>
<li>Turtles cannot distinguish between friendly/enemy mobs or players, but as they are rather impatient, they will try to go around, wait until it has moved or just kill whatever they encounter depending on situation.</li>
<li>Make sure areas where turtles work and move are chunkloaded, with a some room to spare on the sides. Drones do not consider chunkloading at all, so make sure that if a turtle has to take a detour to a target, it won't run into unloaded chunks.</li>
<li>There are situations where navigation and exploration may be more specifically optimized, but you cannot optimize pathfinding for mazes and open space at once - so i have tried to strike a balanced compromise that works in (almost) any situation at decent speed. Pathfinding will try to balance path-lengths, turns-in-a-path, caching and calculation time, trying to determine paths that are not necessarily shortest, but actually fastest from calculation to arrival. A lot of time has been spend on trying out different realistic usage-scenarios and timing different methods within them to come up with this, a little weird, system. Turtles may seem to just randomly run around when exploring an area, but the spots they choose to explore have been chosen with method, to have high efficiency in a large range of situations.</li>
<li>So if your drones are taking hours making their way through the landscape, that may well be normal, depending on the kind of route they have to explore. Complex unknown paths may take hours, easy known paths may take seconds. However, the drones will arrive if there is way, even if it takes days to do so.</li>
<li>Do not give drones coordinates to a target they cannot reach. If you tell a drone to move into a mountain, but don't tell it to do so digging, it will by default try for find a way into that mountain for all eternity, rechecking it over and over.</li>
</ul>

---

##How?
###Setup:
You will need:
<ul>
<li>ComputerCraft</li>
<li>http API enabled in CC configs (only needed for installer and infoserver)</li>
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
(Yes, z is height.)

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

---

##Yeah, ok, but really... why?
To learn lua, to try out some stuff around pathfinding, and to have a playground to experiment with robot-like systems. This was my first foray into lua, to see what that language i already integrated into C++ programs actually is like. 
What a fool i was to ignore it's blissfull simplicity for so long.
