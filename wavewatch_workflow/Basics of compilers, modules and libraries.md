## About nodes, CPUs and cores

A single compute node in a cluster typically contains multiple high-performance CPUs, often combined with RAM and GPUs, making it a powerful computer in its own right. 
While one node can be a "complex" of multiple processor chips and hundreds of cores, it acts as a single, cohesive unit of computation (a "node") within a larger HPC cluster. 
Each CPU chip contains multiple cores (e.g., 32+ cores per CPU), allowing the node to process hundreds of threads simultaneously. All cores on a single node share a common pool of RAM.

- Node: The complete physical machine (the "box").
- CPU (Processor): The physical chip you install in a socket (one node may have 2 CPUs).
- Core: The independent processing unit inside a CPU chip (one CPU may have 64 cores).

## About MPI and OpenMP

The main difference between MPI (Message Passing Interface) and OpenMP (Open Multi-Processing) lies in how they handle memory and how the different processors communicate with each other:

- OpenMP (Shared Memory): All processors (threads) share a single memory space. They can see and modify the same variables directly. It is designed for multi-core CPUs within a single machine.
- MPI (Distributed Memory): Each processor (process) has its own private memory. One process cannot see another's data directly. It is designed to run across multiple computers connected in a network (a cluster).

## What is the difference between modules, libraries and compilers ?

The primary difference lies in their role within the software development process: modules and libraries are the building blocks used to organize and reuse code, while a compiler is the tool that translates that code into a language the computer can understand. 

- Modules: The Smallest Building Blocks 

A module is a single, self-contained unit of code designed to perform a specific task.

Analogy: If you are building a house, a module is a single brick or a specific component like a light switch

- Libraries: The Reusable Toolbox

A library is a collection of pre-written code (often multiple modules or packages) that provides specific functionality so you don't have to build it from scratch.

Analogy: A library is a complete toolbox containing various tools (modules) for a specific job, like a woodworking kit

- Compilers: The Language Translator

A compiler is a special software program that translates human-readable source code into machine code or bytecode that a computer's processor can execute.

Analogy: A compiler is the translator who takes a book written in English and translates the whole thing into a different language for a new audience.

## About compilers

- Check installed compilers and versions: ```dpkg --list | grep compiler```

- list all available compilers that can be installed:
1) ```apt-cache search Compiler```
2) ```apt-cache search Compiler | grep -i --color fortran```

- Install compiler: ```sudo apt-get install gfortran```

```sudo``` command gives you global permission

- found location & version of compilers:
1) ```whereis gcc```
2) ```which gcc```
3) ```gcc --version```

## About modules (kernels)

- check what modules (kernels) are loaded : ```lsmod```
- check how many modules are loaded: ```lsmod | wc -l```
- check a specific module: ```lsmod | grep nvidia```
- check for multiple modules: ```lsmod | egrep -i 'nvidia|e1000e|kvm_intel'```
- get Info about a module: ```modinfo nvidia```

## Checking shared libraries used by executables
Purpose: Verify which system libraries an executable is linked against (useful for debugging runtime errors).

```ldd ww3_shel```

You can also use: nc-config --prefix

