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
<br/>
---

###This system consists of multiple parts:
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
<br/>

---

###Noteable system features:
<ul >
<li>Good pathfinding based on A* with a bunch of optimizations, caching, and shared map. Works well with exploration as well known areas. (Considering turtles are basically blind.)</li>
<li>Mining-functions included in API, like digArea or digChunk (100% quarry pattern, refueling, unloading...)</li>
<li>Drones can specify slots for blocks not be mined, enabling fast mining of all ore while leaving ~60% unwanted untouched and dropping the other.</li>
<li>Binary mapfile format for mapserver, so loading/saving map is faster and takes less space.</li>
<li>Drones are aware of each other, can be individually configured, or have their config overwritten by a job.</li>
<li>More, probably..</li>
</ul><br/>

---

###Known problems:
<ul>
<li>Too many drones will makes everything CC stop working, seems to depend on server-power though.</li>
<li>Lots of bugs!</li>
</ul>

-----

##How?
Jobs can be send by specifying a file or a string containing a valid job.<br/>
Examples:
```lua
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

-----

You have to take care of chunkloading yourself. But, alas, take a look at this: <a href="http://www.computercraft.info/forums2/index.php?/topic/18156-mc-16-cc-158-163-turtle-chunkloaders-mining-chunkloaders-crmod/">CR's turtle peripherals</a>
