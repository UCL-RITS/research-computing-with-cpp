# Boost

## Using Boost

### Introduction

* **[Boost][BoostHome]** is "...one of the most highly regarded and expertly designed ```C++``` library projects in the world."
* A **[large (121+)][BoostDoc]** collection of ```C++``` libraries
* Aim to establish standards, contribute to ```C++11```, ```C++17``` etc.
* Hard to use ```C++``` without bumping into ```Boost``` at some point
* It's heavily templated
* Many libraries header only, some require compiling.


### Libraries included

* Log, FileSystem, Asio, Serialization, Pool (memory) ...
* Regexp, String Algo, DateTime,  ...
* Math, Odeint, Graph, Polygon, Rational, ...

Each has:

* Good documentation
* Tutorial
* Unit tests
* Wide compilation


### Getting started

* Default build system: ```bjam```
* Also ```CMake``` version of boost project, possibly deprecated
* Once installed, many header only libraries, so similar to ```Eigen```


### Installing pre-compiled

* Linux:
    * ```sudo apt-get install boost```
    * ```sudo apt-get install libboost1.53-dev```
* Mac
    * Homebrew (brew) or Macports (port)
* Windows
    * Precompiled binaries? Probably you need to build from source


### Compiling from source

* **[Follow build instructions here][BoostBuild]**
* Or use bigger project with it as part of build system
    * ```Slicer```

### C++ Principles

(_i.e._ why introduce Boost on this course)

* ```Boost``` uses
    * Templates
    * Widespread use of:
        * Generic Programming
        * Template Meta-Programming
    * Functors (see details in old notes)

### ```CMake``` for ```Boost```

* If installed correctly, should be something like:

```cmAKE
set(Boost_ADDITIONAL_VERSIONS 1.53.0 1.54.0 1.53 1.54)
find_package(Boost 1.53.0)
if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIRS})
endif()
```


### ```Boost``` Example

* With 121+ libraries, can't give tutorial on each one!
* Pick one small numerical example
* Illustrate the use of functors in ODE integration


### Using ```Boost``` odeint

* Its a numerical example, as we are doing scientific computing!
* As with many libraries, just include right header

```cpp
#include <boost/numeric/odeint.hpp> 
// Include ODE solver library
// just to check our build system found it
```
* See **[this tutorial][BoostTutorial]**

### Why ```Boost``` for Numerics

* Broader question is
    * Why someone else's library? ```Boost``` or some other.
* Advanced use of Template Meta Programming, Traits
    * Performance optimisations
    * Alternative implementations
        * CUDA via Thrust
        * MPI
        * etc
* You just focus on your bit

[BoostHome]: http://www.boost.org/
[BoostDoc]: http://www.boost.org/doc/libs/1_57_0/
[BoostBuild]: http://www.boost.org/doc/libs/1_57_0/libs/regex/doc/html/boost_regex/install.html
[BoostTutorial]: http://www.boost.org/doc/libs/1_57_0/libs/numeric/odeint/doc/html/index.html
[NumericalRecipesC]: http://www.nr.com/
[WikipediaFunctionPointers]: http://en.wikipedia.org/wiki/Function_pointer
[WikipediaFunctionObject]: http://en.wikipedia.org/wiki/Function_object
