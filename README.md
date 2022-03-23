# Roblox-Integrity-Check-Bypass

# Credits
* [gogo1000](https://github.com/gogo9211)
* [iivillian](https://github.com/iivillian)
* [0x90](https://github.com/AmJayden)

# How to use
Include the utils.hpp and memcheck.hpp file within your dll. Call memcheck::initiate() once before editing any memory.

# How it works
This bypass works by collecting the unseeded hashes of every chunk. The main hasher and active silent hasher are then hooked and the hashes of the chunks within are spoofed with the clean hash calculated against the seed that the server sent.

# Reversal
Roblox integrity check is made of 2 core parts. They have 1 'core' hasher and 16 separate hashers only 1 of which being active along with the 'core' hasher.
For this writeup i will call the core hasher 'main' hasher and rest i will just call hashers

![](https://cdn.discordapp.com/attachments/782842266456817664/956289832290758656/gkp0qufp.png)
![](https://cdn.discordapp.com/attachments/782842266456817664/956290152928526356/jBCw6PAt.png)

The first picture is example of 1 of the 16 hashers and the second picture is of the main hasher. Every single one of those 16 hashers create unique hash so you can't just patch 1 and redirect rest to it, making bypassing it harder.

Only one of them will be active per server and it will be same for everyone in it.
The main hasher is active all the time.

![](https://cdn.discordapp.com/attachments/782842266456817664/956291857787256902/ZAy2wggF.png)

The following function is responsible for calling the active hasher, the arguments passed follows: chunk_start, chunk_size, unknown, an unique value we named seed.
Every time active hasher hash all the chunks, being 30 at this time, this seed will change preventing you from simply saving the value hasher return and keep returning it.

So what we do for those hashers is call the active 1 ourselfs with last argument being 0, which will give us the unique hash and then we save it.
For that we need to get the chunks and their sizes.

```
const auto chunk_o = chunk_start[i];
const auto chunk = m_base - 0x400000 + ((0x1594FE2C * chunk_o - 0x32E7FDBF) ^ (0xD2C21B15 - 0x344B5409 * chunk_o)) + 2;

const auto size_o = size_start[i];
const auto size = (0x1594FE2C * size_o - 0x32E7FDBF) ^ (0xD2C21B15 - 0x344B5409 * size_o);
```

Now all hashers do different operation with this seed so we have function which returns what operation the active hasher use.
Then all we do is hook the hasher and return the unique hash after we do that operation with the seed.

Now for the main hasher things are a bit more complicated.

![](https://cdn.discordapp.com/attachments/782842266456817664/956295870444363796/uETIQD1K.png)

1 chunk is hashed per call to the main hasher, this hash is then xored by the seed and added to the container showen in the picture above.
Seed changes after it hash every chunk ( 30 calls ).
The function does more math with the seed before xoring so we can't use same technique as we do on hashers.

What we did here is set seed to 0, called the function 30 times ( until it hashed every chunk ) and restored the old seed ( so when Roblox call the function themselfs it won't get invalid hash )

If seed is 0 the value it xors by is always 0x68178A72 after they do their math so we can simply do the inverse operation to retrieve a fully unseeded hash of the current chunk and then we store the core hashes too.

Then we just hook both hashers and the main hashers.
