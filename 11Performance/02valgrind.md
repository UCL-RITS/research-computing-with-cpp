---
title: Memory leaks
---

## Checking memory allocation intrusively

There are a variety of compiler flags to check standard memory errors:

- g++: `-fsanitize=address`
- clang: [address sanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)

Good when debugging/testing, but may impact performance. May not detect all
memory errors (e.g. read before initialization).

## Checking memory allocation non-intrusively

[Valgrind](http://valgrind.org/) is an instrumentation framework for Linux and (older) Mac.

It detects memory errors and leaks by intercepting every memory access,
allocation, and deallocation.

Unfortunately, Valgrind currently does not work with Mac OS/X > 10.11.

So let's use docker (on Linux) and docker-machine (Windows, Mac OS/X)!

## Exercise: Traveling salesman solved by Simulated Annealing

Traveling Salesman Problem:

   A salesman living in an n-dimensional world must visit N cities. What is the
   shortest path?

Simulated Annealing:

- Start from a candidate A
- Create a neighbor B of A
- if `path(A) > path(B)`, then swap A and B
- else if `exp(beta * (path(B) - path(A))) > random()`, then swap A and B
- loop until satisfied



### Setting up a docker VM and docker

First download/update to the latest course:

```
git clone https://github.com/UCL-RITS/research-computing-with-cpp
```

Creating a virtual machine is optional on Linux, and necessary on Windows and Mac OS/X

```
> docker-machine create cppcourse  \
           --driver virtualbox      \
           --virtualbox-memory 4000 \
           --virtualbox-cpu-count 2
```

The output from the last command will tell you what expression to run to tell
docker what virtual machine to use.

On Linux and Mac, it will be:

```
> eval $(docker-machine env cppcourse)
```


Then ssh into the machine and look for your home directory:

```
> docker-machine ssh cppcourse
> pwd
# Mac users
> ls /Users/
# Linux users
> ls /hosthome
# Windows users
> ls /c/Users/
```

Then, create a Dockerfile specifying the container we want:

```
> mkdir docker_dir
> cat > docker_dir/Dockerfile <<EOF
FROM ubuntu:latest
RUN  apt-get update && apt-get install -y cmake g++ valgrind
EOF
```

Build an image of the container

```
> docker build -t course_container /path/to/docker_dir
```

Now build  the code in `11Performance/cpp` using an instance (a.k.a container)
of the image.

First, check you can see the directory with the source:

```
> docker run --rm                                               \
          -v /path/to/source/on/vm:/path/to/source/on/container \
          -w /path/to/source/on/container                       \
          course_container                                      \
          ls
```

This should print the content of the directory on your machine, if:

- the virtual box VM was set-up to mount your home directory (automatic)
- the container was set up to mount the VM directory (`-v path/VM/:path/container`)

Finally, replace the ls command to:

1. create a build directory in the source code directory
1. run cmake from the build directory
1. run make in the build directory

## Running valgrind on program called `awful`

Assuming everything went well, there should be a compiled program called awful.

It can only run inside the container!

It has memory leaks and bugs. Investigate and correct using valgrind:

```
> docker run --rm                                         \
    -v /path/to/source/on/vm:/path/to/source/on/container \
    -w /path/to/source/on/container                       \
    course_container                                      \
    valgrind -v --leak-check=full --show-leak-kinds=all   \
                --track-origins=yes ./awful
```

## Running valgrind on program called `less_bad`

Even programs written without explicit memory allocations can have memory bugs.

The next version uses Eigen to solve the same problem.

1. add `libeigen3-dev` to the Dockerfile
1. rebuild the image
1. re-build the code
1. run valgrind on `less_bad`
1. investigate and correct the code
