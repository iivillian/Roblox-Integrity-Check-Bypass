# Roblox-Integrity-Check-Bypass

# Credits
* [gogo1000](https://github.com/gogo9211)
* [iivillian](https://github.com/iivillian)
* [0x90](https://github.com/AmJayden)

# How to use
Include the utils.hpp and memcheck.hpp file within your dll. Call memcheck::initiate() prior to editing any memory.

# About Roblox's Integrity Check


# How it works
This bypass works by collecting the unseeded hashes of every chunk. The main hasher and active silent hasher are then hooked and the hashs of the chunks within are spoofed with the clean hash calculated against the seed that the server sent.

