# A Blast from the Past

2020-06-28
		
Dimitrii Tikhomirov has just [announced a port](https://www.linkedin.com/posts/treblereel_treblereelquake2-gwt-port-activity-6682348732974993408-7toX) 
of our Quake II GWT demo from 2010 to J2CL.

This seems to be the perfect opportunity to finally post a blog entry I have been sitting on for quite while now
-- and to provide some background and history of the port.

## How it all started

The idea to port Quake II to WebGL and GWT was born at a Google internal GWT summit in 2009, where I was presenting a GWT based WebGL 
emulation of the Android Maps for Mobile 3D navigation rendering Engine. I don't remember who brought up the idea, but somebody mentioned 
Jake (the Java port of Quake II) -- and we were immediately thrilled to figure out if it might actually be possible to pull this off.

We knew that this undertaking wouldn't be feasible without significant code changes, as many Java libraries just weren't available in 
the GWT JRE emulation. Some APIs just weren't supported because of memory concerns and others -- such blocking network IO -- were just 
impossible to emulate in GWT in its existing form.

One option would have been to hack the code and just replace anything in Java that isn't available with JSNI. 
But that's not the route we took. If we would have changed the code to no longer compile with regular Java, we'd have lost a 
valuable testing platform that allowed us to check whether we had introduced a bug by converting threaded code to asynchronous 
callbacks -- or in our new graphics routines that map to WebGL. So we decided to go with emulations as far as possible -- and to do 
the conversion from threaded code and blocking IO to async in a way that would still be compatible with regular Java.

For the GL part, one of the problems was that Quake II was still using gl_begin / gl_end style calls, building up rendering data in
a piecemeal approach, which in modern GL has long been replaced with handing over bulk data in one go. This meant that we weren't able 
to use too much of the emulation I had built for Google Maps for Mobile. A full emulation also would need to include texture mapping, 
which in turn required image loading &#8212; which at this point we still needed to be convert to async somehow. So in order to gauge 
feasibility, I went ahead and implemented a simple Canvas2d baed emulation that would just render wireframe images. I can't describe 
how happy I was when I saw the first recognizable Quake II scene rendered with green lines on a black background. It looked a bit 
like the "[Quake on an oscilloscope](youtube.com/watch?v=aMli33ornEU)" demo (which came out later IIRC). I must admit that I am 
still a bit sad that I haven't properly preserved the corresponding rendering code.

Unfortunately, at this point, the frame rate was something in the order of magnitude of 1 FPS, but we managed to improve it quite a 
bit before the April release.


## GWT Debugging and Code Sharing

Still up to today, one of my biggest issues with GWT / J2CL is that System.out.println() doesn't go to the console without a significant 
hack. I am aware that some people like to make fun of "printf debugging", but if you want to compare traces for several platforms, chances
are that parallel tracing with a "real" debugger may get painful -- it's just too easy to step over the significant part of the code.

Also, it's more easy to confirm that the modified source is actually executed and you are not looking at a stale browser cache. 
Now some people might suggest to use a proper logging framework instead of printf, but if debug printing is easily distinguishable 
from logging, it's also much more easy to remove, avoiding excessive log spam left in the code. For instance, at Google we have 
automated checks preventing code with print statements from getting checked in.

One thing I found quite irritating in some GWT libraries was that they provided a replacement for some system feature that 
couldn't be super-sourced for one reason or another -- but then the replacement wouldn't have a matching JRE implementation, 
so even if the corresponding functionality was available in both, GWT and standard Java, the user was left with bridging this
gap in some form or another.

In terms of Java code sharing, there are generally three major options

- Emulate (super-source) a Java API in GWT
- Add a shared abstraction layer with GWT and JRE specific implementations (possible more for Android or J2Objc)
- Emulate a GWT/JS API in "regular" Java</li></ol>

My favorite is option 1, but it's not always possible. Anything related to blocking IO typically can't be emulated in GWT.

I think option 2 is considered the clean default by most people working with shared code, but it has some significant disadvantages:

- It adds an extra layer on all platforms, including the corresponding performance and code size hits
- The corresponding glue code has to be implemented everywhere
- At least some people will know one of the underlying APIs. Why add more mental burden for everyone?

So if option 1 doesn't work, I'd recommend to seriously consider using option 2: Make the Web API available on all platforms. 
This should be even more simple in the age of Jsinterop, where annotations can be used on Java implementations of a web API 
to just point to the corresponding JS classes, methods and properties.

## What happened after the April 2010 Demo?

The last time I checked (for the LibGDX port), some of the code developed for this demo, in particular the Java NIO Buffer 
emulation and the GL emulation, still lived on (to some extend) in the PlayN and LibGDX Java game libraries.

The main problem why we couldn't publicly launch a working web app (opposed to just the source) was that the Quake II 
source was open source, but the levels still are not. The typical way to work around this issue is to let the user provide 
a corresponding "wad" file, which is conveniently extractable from the shareware version that is still available on the web.

After finding a JS zip decompressor, I actually managed to add a download dialog to the WebApp and to launch it on AppSpot.

My next goal was to port the Game to the LibGDX cross-platform game library, as PlayN was never really intended for 
3D games -- and some of the hacks we needed seemed to be an odd fit / maintenance burden for PlayN. In the same go, I wanted to 
check if it the code could work on Android, too. Unfortunately, the GL emulation fork in LibGDX is quite outdated. 
Resource management in LibGDX was built without the web in mind, and the way web support was added later proved to be quite 
incompatible with what we did for Quake. I got a prototype working (replacing the File API deprecated at that time with IndexedDB IIRC), 
but I kind of gave up when the Android built-in zip decode wasn't able to uncompress the shareware game executable. 

Unfortunately, I have never back-ported all my LibGDX changes that were necessary to get the game running, so the LibGDX 
version currently compiles only with my outdated LibGDX fork.

## Learnings -- What I really should have done

Instead of trying to shoe-horn the port into PlayN or LibGDX, where it doesn't really fit well,
I should have pulled out the re-usable parts, in particular the java.nio emulation and the WebGL OpenGL wrapper.

Fortunately, Dimitrii has done this for the [NIO Part](https://github.com/treblereel/gwt-nio) now.

## Links

- <a href="http://quake2playn.appspot.com/">Playable version on AppSpot</a>
- <a href="https://www.linkedin.com/posts/treblereel_treblereelquake2-gwt-port-activity-6682348732974993408-7toX">Dimitrii's announcement</a>
- <a href="http://blog.j15r.com/2010/04/quake-ii-in-html5-what-does-this-really.html">Joel's blog entry</a>
- <a href="http://timepedia.blogspot.com/2010/04/gwtquake-taking-web-to-next-level.html">Ray's blog entry</a>
- <a href="https://github.com/stefanhaustein/quake2-playn-port">Quake II PlayN version source code</a>
- <a href="https://github.com/stefanhaustein/quake2-libgdx-port">Quake II LibGDX version source code</a></li></ul>

