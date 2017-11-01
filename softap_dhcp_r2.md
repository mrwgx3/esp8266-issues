Hi again, devyte

Got my hardware set back up, and finished working through my original notes. Surprisingly, I could reproduce my initial results using the Arduino IDE under Linux Ubuntu 16.04LTS, but not so running under Windows 10 Pro (baffling?!). To eliminate the possibility of an accidently modified SDK and/or ESD damaged hardware, I again re-tested using both the lastest Arduino IDE (version 1.8.5) and new ESP Huzzah hardware, again with the same results.

Now for some backstory...

My initial task was to connect and address some 30+ ESP's together for data collection purposes. Rather than jumping into the complexities of mesh-networking, I explored the possibility of chaining STA+AP configured ESP's into a statically addressed hierarchy. This required that I work in Linux in order to easily modify and recompile the required SDK libraries.

I turned on debugging using IDE's generic ESP settings and got things partially working, but everything broke when I turned debugging OFF. Testing via 'ifconfig' and 'ping' showed that the DCHP addressing being handed out by the ESP access point were nothing close to those requested, hence this issues post.

Now back to the present...

I agree with you that the status of the debug flags should make no difference on how the code performs. Apparently this is now the case for the IDE running under Windows 10; the code generates erroneous DCHP addresses regardless of debug flag state. This makes sense given what the code contains!

Running under Ubuntu Linux, however, the assigned DCHP address DO depend on debug flag status. Even more disturbing is that

* Correct addresses are calculated when debug 'printf' statements are enabled; how and where is this happening?
* One can cause the erroneous DCHP addresses to reappear by changing the number, hence argument depth of the debug printf statements being used.

This odd behavior does imply a memory corruption issue, but at how deep a level is it occuring at?

Given that...

* The debug printf statements seem to ultimately call a 'vnsprint()' function in the closed SDK, and
* The interrupt intensive nature of the Wifi stack,

I suspect that only detailed knowledge of the closed SDK plus a true hardware emulator will resolve the problem.
I think this issue should be forwarded onto Espressif, as it potentially impacts so many code modules.

