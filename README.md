# CompilingOnAWSLambda
July 2018 DevOpsDSM talk



Information Theory 101
![In this house we obey the laws of thermodynamics!](https://pbs.twimg.com/media/DOjUx5BWsAAwR9h.jpg)
* System Latency: How long does a system take to respond to a resquest?
* Distance Latency : How far does information have to travel? Fastest is 1 foot per nanosecond (speed of light). 
* Bandwith: How much information can be transered in a unit of time.
* Transfer time =  SystemLatency + (Distance * DistanceLatency) + (FileSize / Bandwidth)
* Random File: There exists no program that will emit the file that has shorter length of the file itself
* Logarithm Base K: How many times you can chop a log into K equal portions until you reach a portion of unit size.
* Size of counters, depth of K-ary trees: Logarithm Base K
![Logarithm visual](https://www.garrettwade.com/media/catalog/product/cache/1/image/730x/0dc2d03fe217f8c83829496872af24a0/2/0/20f0101-westernlogsaw-web-0131_c_r.jpg)





What is AWS Lambda?
* Amazon Linux Container
* Executes a static zip file up to 50MB with a small user supplied JSON payload
* Runtimes in supported languages, but you can shell out or dynamically load any native binary.
* Max 5 minute runtime
* May download and execute more code from the network
* Charged per 1/10th second used. Large startup latency for Java SDK, lowest for native Go SDK.
* Throttled by memory allocated
* /tmp folder for scratch local storage
* Intel Xenon. Beware, may execute on different generations breaking new SIMD instructions


Most common build sytems:
* Make - Stuart Feldman at Bell Labs (1976) language agnostic
* Excel - Microsoft (1993) cell based script evaluation
* CMake -  (2000) - Emits Make, XCode, Ninja, Visual Studio, ... 

Java Build Systems:
* Ant (2000) - Based on Make 
* Maven (2004) - Less XML 
* Gradle (2007) - Build rules in Groovy
* SBT (2008) - Scala/Java

Newest kids on the block:
* DistCC (2002) - TCP/SSH syncronized parallel builds
* Ninja (2012) - Developed for Google Chrome
* Buck (2013) - Facebook 
* Bazel (2015) - Google [Refused to support AWS yesterday](https://github.com/bazelbuild/bazel/pull/4889)
* Webpack, Grunt, Gulp, Yeoman - Javascript shiny...


Universal concepts:
* Directed Acyclic Graph of build dependencies
* Scheduling the DAG

Non-determinism and impurity:
* Direct reliance on system state (timestamps ...)
* Benchmark/Integration test results that rely on system state
* Pulling in depedencies instead of pinning them

Bottlenecks:
* Must obey the laws of physics and pay the latency and bandwith costs of moving files
* Context Free Grammar Parsing, [Complexity of sparse matrix-matrix-multiply - Valiant 1975](https://arxiv.org/abs/cs/0112018)
* Linking object files is usually linear complexity
* Macro/Template application can be costly

Efficencies:
* Keep track of artifacts that do not need rebuilt
* Artifacts can be hashed to infer changes (Git for example)
* Files like system headers may be available on build nodes and do not require network transfer
* Use forward declares of types instead of includes to minimize duplicate compliation
* Pre-process files from text to binary in-memory format
* Use mutual informaton of dependinces for file compression [Vitanyi 2003](https://arxiv.org/abs/cs/0312044)

The process:
* Intercept/log calls to compiler and linker
* Record compile times for each build step
* Record file sizes
* Build artifact graph
* Solve for build plan


Recompiles:
* Cache precompiled headers (larger file size, lower processing time)
* Cache LLVM IR 
* Refactor slowly building files. (Do you really need that complex template?)
* Don't be like Megacorp and use Russian doll artifacts. Flat trees, not linear order of artifacts.
* Strip unused includes [Include what you use](https://github.com/include-what-you-use/include-what-you-use)


"-MD" both compiles and emits makefile

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



Scratch notes:

[Dotfile generation may be broken on the latest Clang](https://stackoverflow.com/questions/51416847/clang-usage-of-dependency-dot-option/51428076#51428076)







