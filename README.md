# CompilingOnAWSLambda
July 2018 DevOpsDSM talk



Parallel builing at scale


Latency: How long do you have to wait to establish get any information back. (Speed of light bounded)
Bandwith: How much information can be transered in a unit of time.

Information transfer time = Latency + size/bandwidth


Make - Stuart Feldman at Bell Labs (1976) language agnostic
Excel - Microsoft (1993) cell based script evaluation

CMake -  (2000) - (Make, XCode, Ninja Visual Studio) 

Ant - (2000) Java
Maven - (2004) Java
Gradle - (2007) Java 
SBT - (2008) Scala/Java

DistCC (2002) - TCP/SSH syncronized
Ninja (2012) - Developed for Google Chrome
Buck - (2013) Facebook 
Bazel - (2015)  Google limited feature open source of their Blaze build system (C,C++,Java,Go, Python, Objective-C, Bash)

Webpack, Grunt, Gulp, Yeoman - Javascript ...


Universal concepts:
A Directed Acyclic Graph of build dependencies.
Scheduling the DAG

Non-determinism:
Timestamps
Benchmark/Integration test results

Bottlenecks:
Context Free Grammar Parsing  (Complexity of sparse matrix-matrix-multiply - Valiant 1976)
Linking - usually linear
Macro/Template application - Varies

Efficencies:
Hash large artifacts to infer membership



* Intercept/log calls to compiler and linker
* Record compile times for each build step
* Build artifact graph. Annotate with file sizes and compile times
* Solve for build plan

Recompiles:
Cache precompiled headers (larger file size, lower processing time)
Cache LLVM IR 
Refactor slowly building files. (Do you really need that complex template?)
Don't be like Megacorp and use a Russian doll artifacts. Flat trees, not linear builds.

Strip unused includes:
https://github.com/include-what-you-use/include-what-you-use


"-MD" both compiles and emits makefile

Issue with dotfile generation
https://stackoverflow.com/questions/51416847/clang-usage-of-dependency-dot-option/51428076#51428076

[Open source tools for examining include dependencies](http://gernotklingler.com/blog/open-source-tools-examine-and-adjust-include-dependencies/)

[GCC command line flags](https://github.com/gcc-mirror/gcc/blob/274d31f044ac1c4610b67d2220237f0387aa367f/gcc/c-family/c.opt)

Step 1: Generate the JSON compilation database


[JSON Compiliation Databases](http://clang.llvm.org/docs/JSONCompilationDatabase.html)

```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=YES ..
# Creates the file compile_commands.json
```
Entries in the JSON compilation database look like this:
```json
{
  "directory": "/Users/crb002/github/codeanalysis/libpng/build",
  "command": "/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc -DPNG_INTEL_SSE_OPT=1 -I/Users/crb002/github/codeanalysis/libpng/build -I/Users/crb002/github/codeanalysis/libpng    -o CMakeFiles/png_static.dir/pngwrite.c.o   -c /Users/crb002/github/codeanalysis/libpng/pngwrite.c",
  "file": "/Users/crb002/github/codeanalysis/libpng/pngwrite.c"
},
```


Step 2: 
Annotate with clang -H to get header files
Annotate with timings to get compilation times for each entry

Step 3:
Build the dependency graph
Annotate with file sizes

Step 4:
Put annotated dependency graph into SMT solver, get a build plan.

Step 5: 
Compile






