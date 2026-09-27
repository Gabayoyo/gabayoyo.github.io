---
title: "Aaron Gabayoyo"
lede: "About me"
description: "The about me section of Aaron Gabayoyo's homepage"
author_profile: true
permalink: /
---

Hey, my name's Aaron and I'm a final-year MInf (Master of Informatics) student at the University of Edinburgh. Most of my work is in anything compilers, ranging from surrounding tooling to direct optimisation, especially within the LLVM/MLIR ecosystem. I especially love engineering compilers in such a way to maximise the performance potential from hardware and software. Essentially, big number go down.

In my fourth year, i worked on my honours project: [an extensible IR interpreter](https://github.com/edin-dal/scair/blob/main/interpreter/README.md) for the ScaIR compiler framework. By constructing an MLIR-compatible interpreter, developers are able to prototype their dialects by specifying high-level operation implementations without needing to worry about lowering details or the interpretation specifics. Crucially, the ScaIR interpreter was able to achieve a 12-18x speedup over xDSL's IR interpreter while needing less LoC on average.

This year, I'll be researching another honours project, this time on implementing the Chisel hardware synthesis pipeline using ScaIR to remove the CIRCT C++/MLIR dependency and investigating the potential benefits that come with that.

During my research internship with MLSystems CDT, supervised by Prof. Ajitha Rajan, I built [mlir-mracle](https://github.com/dakaidan/MLIR-MRacle), a metamorphic testing harness for the MLIR OpenMP dialect. It has two key novelties: concurrency-safe fuzzing and memory-model-aware transformations, with both enabling methods of testing previously unapplicable for concurrent programs in MLIR. That combination led to the discovery of missing OpenMP verification logic for nested `omp.critical` ops, which was [amended and merged into MLIR upstream](https://github.com/llvm/llvm-project/pull/217357). You can read more about this project in this [short report here](/files/mlir_mracle.pdf).

Recently, i've developed an MLIR dialect, [mlir-match](https://github.com/Gabayoyo/mlir-match), dedicated to representing functional pattern constructs as MLIR operations. This, in turn, allows for the application of Luc Maranget's algorithm, converting flat pattern match structures into decision trees and reducing the number of comparisons by 4-15x and runtime by up to 3.04x on average.

I don't just work on compilers either:
- I built [BiggerBrother](https://github.com/Gabayoyo/BiggerBrotherWebApp), a computer vision form analysis software that derives helpful metrics from recorded sets, including the world's first proximity to failure estimator from fitting a personalised load-velocity curve. It also comes with a rigorous CI pipeline: 121 tests across unit, integration and end-to-end tiers, with ruff linting, mypy type-checking and pytest running every push.
- I implemented [RISC-E](https://github.com/Gabayoyo/RISC-E), a C++ harness for prototyping RV32I hardware. Using its own RISC-V ELF loader and interpreter as a base, hardware components and logic can easily be defined and redefined at the C++ level instead of using a HDL. More importantly, RISC-E allows for simultaneous comparison of components through simulated cost models, letting developers gauge possible performance differences at a cheap cost.

I am also interested in anything low-level (not just compilers), particularly interpretation, robotics and FPGA/hardware synthesis. 

Feel free to contact me @ aarongaba05@gmail.com if you wanna talk compilers or anything else. :)
