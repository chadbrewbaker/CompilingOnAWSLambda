# CompilingOnAWSLambda
July 2018 DevOpsDSM talk

This talk is about parallel "serverless" builds. Concrete examples are Linux and C++ but feel free to ask about other build toolchains.

Information Theory 101
![In this house we obey the laws of thermodynamics!](https://pbs.twimg.com/media/DOjUx5BWsAAwR9h.jpg)
* System Latency: How long does a system take to respond to a resquest?
* Distance Latency : How far does information have to travel? Fastest is 1 foot per nanosecond (speed of light). 
* Bandwith: How much information can be transered in a unit of time.
* Transfer time =  SystemLatency + (Distance * DistanceLatency) + (FileSize / Bandwidth)
* Random File: There exists no program that will emit the file that has shorter length of the file itself
* Logarithm Base K: How many times you can chop a log into K equal portions until you reach a portion of unit size.
![Logarithm visual](https://www.garrettwade.com/media/catalog/product/cache/1/image/730x/0dc2d03fe217f8c83829496872af24a0/2/0/20f0101-westernlogsaw-web-0131_c_r.jpg)
* Size of counters, depth of K-ary trees: Logarithm Base K


I don't want to compile the compiler.
* [Use Lambdamart GCC for AWS Lambda](http://www.lambdamart.com)
* Reach out to Lambdamart if you want another build chain supported

I want serverless command line tools, but I don't want to host them.
* Lambdamart tools through RapidAPI and AWS Marketplace are in beta.
* Reach out to Lambdamart for enterprise hosting
* Lambdamart can crush Hadoop [235x faster than Hadoop on one node](https://adamdrake.com/command-line-tools-can-be-235x-faster-than-your-hadoop-cluster.html)
```bash
# Set up Lambamart API key and endpoint
# Set up method to authorize temporary S3 credentials
lambdamart sort s3://MyBucket/foo.txt -o s3://myOtherBucket/fooSorted.txt
lambdamart gfactor 234234234
# 234234234: 2 3 3 3 13 333667
lambdamart build s3://MyBucket/source s3://MyBucket/artifacts
```


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
* Make (1976) - Stuart Feldman at Bell Labs, language agnostic
* Excel (1993) - Microsoft cell based script evaluation
* CMake (2000) - Emits Make, XCode, Ninja, Visual Studio, ... 

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
* Webpack, Grunt, Gulp, Yeoman - Javascript shiny


Universal concepts:
* Directed Acyclic Graph of build dependencies
* Scheduling the DAG

Non-determinism and impurity:
* Direct reliance on system state (timestamps ...)
* Benchmark/Integration test results that rely on system state
* Pulling in depedencies instead of pinning them

Bottlenecks:
* Must obey the laws of physics and pay latency and bandwith costs
* Context Free Grammar Parsing, [Complexity of sparse matrix-matrix-multiply - Valiant 1975](https://arxiv.org/abs/cs/0112018)
* Linking object files is usually linear complexity
* Macro/Template application can be costly

Efficencies:
* Keep track of artifacts that do not need rebuilt
* Artifacts can be hashed to infer changes (Git for example)
* Files like system headers may be available on build nodes and do not require network transfer
* Strip unused depednencies [Include what you use](https://github.com/include-what-you-use/include-what-you-use)
* Use forward declares of types instead of includes to minimize duplicate compliation
* Pre-process files from text to binary in-memory format
* Use mutual informaton of dependinces for file compression [Vitanyi 2003](https://arxiv.org/abs/cs/0312044)
* Refactor slowly building files
* Use sparse linking so you don't have to include entire large libraries
* Don't be like Megacorp and use Russian doll artifacts. Flat trees, not linear order of artifacts.

Plan of attack:
* Intercept calls to compiler and linker
* Record compile times for each build step
* Record file sizes
* Estimate bandwidth/latency costs for your build nodes
* Build artifact graph
* Solve for build plan

AWS Lambda Specific Issues:
* Containers are extremely lean, have to package missing library dependencies
* Containers change contents without much notice, must Hoover files from live container
* Store dependencies in S3, load them to /tmp/ at Lambda startup
* GCC is larger than 50MB without taking an axe to it, you have to load it from S3

"-MD" both compiles and emits makefile

[Open source tools for examining include dependencies](http://gernotklingler.com/blog/open-source-tools-examine-and-adjust-include-dependencies/)

[GCC command line flags](https://github.com/gcc-mirror/gcc/blob/274d31f044ac1c4610b67d2220237f0387aa367f/gcc/c-family/c.opt)

## Step 1: Generate the JSON compilation database


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


## Step 2: Use the compiler to extract file dependencies and estimate compile times
```bash
clang -H foo.c
# Will extract all header files used by foo.c
```
```bash
gtime clang foo.c
# Optionlly collect the time for each artifact build
```

## Step 3: Build the dependency graph
* Python script

Step 4: Solve for a build plan
* Z3 SMT Solver
* Python bindings if you are allergic to C or Lisp interfaces


Scratch notes:

[Dotfile generation may be broken on the latest Clang](https://stackoverflow.com/questions/51416847/clang-usage-of-dependency-dot-option/51428076#51428076)







