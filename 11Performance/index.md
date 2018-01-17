---
title: Post-coding medley, memory leaks, and performance measurements
---

## Please install docker and docker-machine

Mac OS/X:

```
> brew install Caskroom/cask/virtualbox
> brew install docker-machine
> brew install docker
> brew install qcachegrind
```

or

```
> brew install Caskroom/cask/docker-toolbox
```

Linux:

- docker https://www.docker.com/community-edition
- docker machine (optional): https://docs.docker.com/machine/install-machine/
- qcachegrind or kcachegrind

Windows or Mac OS/X:

- https://www.docker.com/products/docker-toolbox

## Side-note: Flynn's Taxonomy of Parallelization

- SISD: Single instruction single data

  prototypical serial code

- SIMD: Single instruction multiple data

  Same instruction is performed in parallel over different inputs. Necessary
  in GPU (at the level of a warp of 32 threads). Likely in OpenMP
  (for loop parallelization) and MPI.

- MIMD: Multiple instruction multiple data

  Basically, different threads or different nodes doing different things, e.g.
  computing different terms in an equation, dealing one with the GUI, the other
  with a database, etc...

- MISD: Multiple instructions single data

  Weird... Used for fault tolerance (different algorithm that should lead to
  same output).

## All tests pass, the code works: are we done yet?

Lots of changes can be made to a code, from cosmetic to crucial:

- formatting, linting, and refactoring
- checking for memory leaks
- profiling and performance
- benchmarking
