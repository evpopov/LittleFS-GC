# LittleFS-GC
A garbage collector for LittleFS

**This is a work in progress. Please be patient. I will push the initial release next week.**

# Background
[LittleFS](https://github.com/littlefs-project/littlefs) is a popular embedded filesystem. One of the design decisions was to only delete blocks as they are needed. Additionally, sometimes blocks need to be relocated which requires deletion and copying of data. Both of these result in many users getting frustrated with the big performance degradation they see once the filesystem gets "dirty" enough to require relocation. I am one of these users, but I really like LittleFS, so this is my attempt to remedy the situation.

# My Workaround
There is nothing I can do about the need for relocation, so my approach is to simply make writing and especially relocation as fast as humanly possible.
The solution is quite simple actually:  
LittleFS assumes that a block needs to be erase before it writes to it for the first time. Erasing flash is slooooow, so this garbage collector modifies this default behaviour. Blocks only get erased if they are known to be dirty. Otherwise the new code goes straight to writing. As long as the file system is kept clean, writing to it is fast and fairly predictable. Even relocation ( when it happens ) is barely noticable.

# How it works 
The garbage collector keeps track of which sectors are used, free and erased, or dirty ( not used, but not confirmed to be erased )
