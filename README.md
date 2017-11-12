CONTENTS
========
* Introduction
* How to install the framework
  - Prerequisites
  - Installation
* How to run the framework
  - Benchmarks
*Experiments from the PLDI'15 paper

Contact for any related enquiries: Yousun Ko (yousun.ko@cs.yonsei.ac.kr)

INTRODUCTION
============
LaminarIR is a lower-level intermediate representation (IR) for stream 
programming. In LaminarIR, named tokens are used for data communication 
among nodes instead of FIFO (first-in, first-out) semantics. This new 
approach shifts the FIFO buffer management from run-time to compile-time and
provides sufficiently low abstraction of dataflow for LLVM to promote all
token variables to SSA form.
This archive contains the LaminarIR framework and the benchmarks for
performance evaluation.


HOW TO INSTALL THE FRAMEWORK
============================
Prerequisites
-------------
Prerequisite installations for the LaminarIR framework is listed below:
- JDK
- python
- pygraph 
- StreamIt
- Clang & LLVM
- Likwid
- PAPI
- ANTLR v3.1

JDK is needed to compile the syntax rule of the LaminarIR. JDK is one of
the prerequisites to install StreamIt as well.
Python and pygraph are required to parse LaminarIR and generate c codes.
StreamIt is required to evaluate StreamIt for comparison.
Clang and LLVM are required to compile C codes and generate compiler 
optimization statistics.
Likwid and PAPI are required to improve measurement quality. Likwid enables
to pin processes on specific cores and PAPI provides a wrapper to access 
hardware performance counters.
ANTLR v3.1 is required to generate LaminarIR parser. Download the ANTLR v.3.1
jar file and locate under src/bin.

Installation
------------
The LaminarIR compiler needs to be parameterized for the underlying 
hardware architecture. This setting is kept in `laminar/src/configure'.
Set the `MACHINE' variabel according to the underlying host CPU.
Then run the configure script:

./configure

This process will generate Defines.make which defines machine specific 
compilation settings for code compliation and compile the whole framework.


HOW TO RUN THE FRAMEWORK
========================
The framework provides a python script `laminar/src/autorun.py', to 
(re)configure the system, compile and execute each benchmarks, and collect 
the measured results at once. Running `autorun.py' will invoke each steps 
below in order.
1. Configure current processor architecture setting given in `configure'
   and set the compiler options accordingly.
2. Generate LaminarIR direct access format or FIFO queues from a 
   high-level stream programming language. LaminarIR frontend for StreamIt 
   is provided which reads StreamIt source codes from 
   `laminar/src/examples/streamit/strs'.
3. Generate C code on LaminarIR direct access format or FIFO queues by 
   reading LaminarIR codes under the benchmark directories e.g.,
   `laminar/src/examples/laminarir/direct'. Or reads generated C++ codes
   by StreamIt-2.1.1 compiler stored under `laminar/src/examples/streamit/instrumented'.
4. Compile and run the obtained C/C++ code with clang.
5. Collect the measured performance and append to 
   `laminar/src/papi_events.csv'
6. Backup all the by-product codes and measured performance under 
   corresponding directory under `laminar/src/results' e.g., 
   `laminar/src/results/laminar_direct' or 
   `laminar/src/results/streamit'.

Find `laminar/src/report.log' if you want the logs of commands executed 
by the `autorun.py' script.


Benchmarks
----------
The LaminarIR compiler provides two backends, one for the LaminarIR 
direct-access format, and one for FIFO queues. All benchmarks from the 
experimental evaluation of the conference paper are provided, i.e., 
DCT, DES, FFT2, MatrixMult, AutoCor, Lattice, Serpent, JPEGFeed, BeamFormer,
ComparisonCounting, and RadixSort.
LaminarIR code in direct token-access is provided for each benchmark in,
  `laminar/src/examples/laminarir/direct'. 
LaminarIR code employing FIFO queues is provided for each benchmark in,
  `laminar/src/examples/laminarir/fifo'.
The C++ code of each benchmark as generated by StreamIt-2.1.1 compiler is found 
under,
  `laminar/src/examples/streamit/instrumented'

For each benchmark, two versions are provided: the original StreamIt 
benchmark code, plus one version that we created for using randomized input
(the original StreamIt benchmarks use (mostly) static input). A file name
`<name>.rand.*' denotes code with randomized input, and `<name>.*' denotes
code with static input.


Experiments from the PLDI'15 paper
==================================
Case 1. LaminarIR code generation (Fig. 3)
------------------------------------------
1. Running following command under `laminar/src/frontend' will generate
LaminarIR direct-access format (or FIFO queues with `--laminar fifo' option) 
with static inputs (or random inputs with `--random-input') under 
`laminar/src/frontend/compile' by reading StreamIt codes from 
`laminar/src/examples/streamit/strs'.

  python LaminarIRGen.py --laminar direct --random-input DCT

Case 2. Performance evaluation (Fig. 4 & Tab. 3 from the paper)
---------------------------------------------------------------
1. Running following command under `laminar/src' will evaluate LaminarIR
direct-access format, FIFO queues and StreamIt over all benchmarks and
generate a `papi_events.csv' which contains:
execution times, clock cycles per introduction, number of loads and stores,
energy consumption of each CPU of each benchmark if the corresponding
hardware performance counters are provided by the machine that the framework
runs on.

  python autorun.py --laminar direct --laminar fifo --streamit

2. If you want to select a specific benchmark to run, append name of the
benchmark to the command e.g.,

  python autorun.py --laminar direct --laminar fifo --streamit DCT

3. You can be selective on the code versions to test e.g.,
 
  python autorun.py --laminar direct --streamit DCT

will run DCT on LaminarIR direct-access format and StreamIt only.

4. Append `--generate-LaminarIR streamit' option if you want to generate 
LaminarIR code from the LaminarIR frontend for StreamIt programs. Otherwise, 
the Framework will read LaminarIR codes from `laminar/src/examples/laminarir'.

Case 3. Effectiveness of the LaminarIR direct access format with compiler 
optimizations (Fig. 5 from the paper)
-------------------------------------------------------------------------
1. Run Case 2 to get execution times with random inputs.
2. Run Case 2 again with `--static-input' to get execution times with 
static inputs e.g., 

 python autorun.py --laminar direct --laminar fifo --streamit --static-input

3. Calculate 

  (Execution time with static inputs)/(Execution time with random inputs)

to obtain effectiveness of each LaminarIR direct-access format, FIFO queues
and StreamIt with compiler optimizations.

Case 4. Improvement rate of each LLVM optimization passes (Fig.6 from the 
paper)
-------------------------------------------------------------------------
1. Append `-O' or `--opt-test' to Case 2, if you want to evaluate each of
LLVM optimization passes e.g., 

 python autorun.py --laminar direct --laminar fifo --streamit --opt-test DCT

Note: This procedure can take several hours for a benchmark.

Case 5. Communication reduction rate (Tab. 4 from the paper)
------------------------------------------------------------
1. Run below command under `laminar/src' to get ``total communication in 
byte'' and ``eliminated communication in byte'' after exploiting LaminarIR
direct access format per steady state iteration. E.g.,

  python laminar.py --elim_comm examples/laminarir/direct/DCT.sdf

will give you the total communication in byte and the eliminated 
communication in byte of DCT along with numbers of actors and splitjoins 
etc. Note that the full path to the code should be given unlike the other
cases.

Case 6. SSA promotion count (Tab. 5 from the paper)
---------------------------------------------------
1. Append `-a' or `--llvm_analyzer' to Case 2, if you want to get LLVM 
compiler statistics e.g., 

  python autorun.py --laminar direct --laminar fifo --streamit --llvm_analyzer

It will generate `<name>.O3.stats' in addition which contains `Number of 
allocas promoted to SSA values'.

