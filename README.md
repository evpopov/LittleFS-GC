# LittleFS-GC
A garbage collector for LittleFS 

***NOTE: Instead of supplying a set of patch files, I decied to pull in all the original code and keep rebasing this project on top of the upstream repository.***

# Background
[LittleFS](https://github.com/littlefs-project/littlefs) is a popular embedded filesystem. One of the design decisions was to only delete blocks as they are needed. Additionally, sometimes blocks need to be relocated which requires deletion and copying of data. Both of these result in many users getting frustrated with the big performance degradation they see once the filesystem gets "dirty" enough to require relocation. I am one of these users, but I really like LittleFS, so this is my attempt to remedy the situation.

# My Workaround
There is nothing I can do about the need for relocation, so my approach is to simply make writing and especially relocation as fast as humanly possible.
The solution is quite simple actually:  
LittleFS assumes that a block needs to be erase before it writes to it for the first time. Erasing flash is slooooow, so LittleFS-GC modifies the default behaviour. Blocks only get erased if they are known to be dirty. Otherwise the new code goes straight to writing. As long as the file system is kept clean, writing to it is fast and fairly predictable. Even relocation ( when it happens ) is barely noticable.

# How it works 
- Upon mounting, the garbage collector creates a map of the block in the file system. It keeps track of which sectors are used, free and erased, or dirty ( not used, but not confirmed to be erased ). 
  - LittleFS traverses the filesystem and records all used blocks.
  - It then updates its internal map to mark all other blocks as dirty.
  - Next, the GC reads all free blocks. If a block is confirmed to be truly clean, it is marked as such in the internal map.
  - Lastly, and optionally, the GC locates the first block that LittleFS will write to and proactively cleans up a configurable number of blocks. This optional step ensures that whenever LittleFS attempts to write for the first time, it will hit a sequence of free and confirmed clean blocks.
- In the background, the GC does the following:
  - Traverses the filesystem to update its used block map.
  - Systematically, cleans all dirty blocks.
- LittleFS is then modified to only call the user-supplied erase functino if the GC has that block marked as dirty.

LittleFS-GC uses bitmaps to keep track of used and free blocks.
