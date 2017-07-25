# apriltagJS

Apriltag port to JS using Emscripten.

https://github.com/terraswarm/apriltagJS

This is an attempt at porting the [Apriltag](https://april.eecs.umich.edu/software/apriltag.html) visual fiducial system from C to JavaScript using Emscripten.

Sadly, the result is very slow, in part because the Emscripten output is single threaded.

One possible solution is to use lower resolution images.

## How to build

1. Download [Emscripten](http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html)

2. View the README.md and install. Under Mac:
```
    ./emsdk update
    ./emsdk install latest
    ./emsdk activate latest
    source ./emsdk_env.sh
```

3. Optional: Create a file and compile it:
```
    bash-3.2$ cat main.c
    #include <stdio.h>

    int main() {
      printf("hello, world!\n");
      return 0;
    }
    bash-3.2$ emcc main.c
    INFO:root:generating system library: libc.bc... (this will be cached in "/Users/cxh/.emscripten_cache/asmjs/libc.bc" for subsequent builds)
    INFO:root: - ok
    INFO:root:generating system library: dlmalloc.bc... (this will be cached in "/Users/cxh/.emscripten_cache/asmjs/dlmalloc.bc" for subsequent builds)
    INFO:root: - ok
    bash-3.2$ node a.out
    hello, world!
    bash-3.2$
```

4. Build in apriltag:
```
    cd apriltag
    make
```

5.  Run an example while in the apriltag/ directory:
```
    bash-3.2$ ./example/apriltag_demo ../tag36_11_00586.jpg 
    loading ../tag36_11_00586.jpg
    pjepg: Unknown marker ee at offset 0f56
    detection   0: id (36x11)-586 , hamming 0, goodness    0.000, margin  119.062
     0                             init        0.008000 ms        0.008000 ms
     1                       blur/sharp        0.001000 ms        0.009000 ms
     2                        threshold        4.221000 ms        4.230000 ms
     3                        unionfind        8.689000 ms       12.919000 ms
     4                    make clusters        3.575000 ms       16.494000 ms
     5            fit quads to clusters       22.117000 ms       38.611000 ms
     6                            quads        1.430000 ms       40.041000 ms
     7                decode+refinement        1.049000 ms       41.090000 ms
     8                        reconcile        0.002000 ms       41.092000 ms
     9                     debug output        0.002000 ms       41.094000 ms
    10                          cleanup        0.011000 ms       41.105000 ms
    hamm     1     0     0     0     0     0     0     0     0     0       41.097     1
    Summary
    hamm     1     0     0     0     0     0     0     0     0     0       41.097     1
    bash-3.2$ 
```

6.  Use Emscripten to build the JS version:

```
    make apriltags_demo.js 
```

7.  Run an example:

```
    bash-3.2$ node apriltags_demo.js /working/tag36_11_00586.jpg
    Arg: /working/tag36_11_00586.jpg
    enlarged memory arrays from 16777216 to 67108864, took 18 ms (has ArrayBuffer.transfer? false)
    Warning: Enlarging memory arrays, this is not fast! 16777216,67108864
    loading /working/tag36_11_00586.jpg
    pjepg: Unknown marker ee at offset 0f56
    enlarged memory arrays from 67108864 to 134217728, took 36 ms (has ArrayBuffer.transfer? false)
    Warning: Enlarging memory arrays, this is not fast! 67108864,134217728
    detection   0: id (36x11)-586 , hamming 0, goodness    0.000, margin  119.062
     0                             init        1.000000 ms        1.000000 ms
     1                       blur/sharp        0.000000 ms        1.000000 ms
     2                        threshold      467.000000 ms      468.000000 ms
     3                        unionfind       82.000000 ms      550.000000 ms
     4                    make clusters      467.000000 ms     1017.000000 ms
     5            fit quads to clusters      418.000000 ms     1435.000000 ms
     6                            quads      976.000000 ms     2411.000000 ms
     7                decode+refinement      190.000000 ms     2601.000000 ms
     8                        reconcile        1.000000 ms     2602.000000 ms
     9                     debug output     2655.000000 ms     5257.000000 ms
    10                          cleanup        1.000000 ms     5258.000000 ms
    hamm     1     0     0     0     0     0     0     0     0     0     5257.000     1
    Summary
    hamm     1     0     0     0     0     0     0     0     0     0     5257.000     1
    bash-3.2$ 
```

8.  Note that the JavaScript version using Emscripten is over 100x slower than the C version.

## Changes that were made

1. The getopt library was not well supported, so that code is commented out in `apriltag/example/apriltag_demo.c`

2. To pass files in to code compiled with Emscripten, we need to define use the [Emscripten Filesystem API](https://kripken.github.io/emscripten-site/docs/api_reference/Filesystem-API.html).  So, we define a path called `/working`, so we add the following to apriltag/example/apriltag_demo.js

```
    #ifdef JS
    #include <emscripten.h>
    #endif
    
    int main(int argc, char *argv[])
    {
    
    #ifdef JS
      EM_ASM(
             FS.mkdir('/working');
             FS.mount(NODEFS, { root: '.' }, '/working');
             );
    #endif
```

3. [Emscripten has pthreads support](https://kripken.github.io/emscripten-site/docs/porting/pthreads.html) but it only works with FireFox Nightly channel: "Any code that is compiled with pthreads support enabled will currently only work in the Firefox Nightly channel, since the SharedArrayBuffer specification is still in an experimental research stage before standardization. "
So, if apriltag_demo.c is modified so that only one thread is used:

```
    td->nthreads = 1;
```

For more information, see https://www.icyphy.org/accessors/wiki/Notes/AprilTags


