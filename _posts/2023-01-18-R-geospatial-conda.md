---
layout: post
title: "Geospatial R and Conda"
tags: R, conda
---

[Conda]() is a great package manager and virtual environments tool that helps users take ownership of managing their software environments.
I'm a big fan of Conda and use it myself to manage Python and R installations for personal projects.
It's also a tool we use and recommend to people using our [HPC systems] at Leeds, as you can't `apt install` things as users on ARC3/ARC4 due to the required privilieges.

I was looking at helping a user install some R geospatial packages yesterday that had some library requirements that didn't align with versions we had installed on our HPC.

So I thought, fine, let's handle this via Conda, create ourselves a new environment with a fresh new R install and some of these geospatial libraries, everything will install fine, so I thought...

To begin with I wanted to install [`raster`](https://cran.r-project.org/web/packages/raster/index.html) (which I note has been superseded by [`terra`](https://cran.r-project.org/web/packages/terra/index.html)). 
And `terra` depends on GDAL, GEOS, PROJ and needs a C++11 compiler handy. 
As a starter for 10 I went with the following Conda command:

```bash

$ conda create -n r-geos -c conda-forge r-base=3.6.1 gdal

```

Great, i've got my R installation of the version I need and i've pulled in the geospatial libraries i'm after. 
Let's go and install `raster` and it's dependencies!

```bash
$ source activate r-geos

(r-geos) $ R -e 'install.packages("raster");'
```
```output

```

And here is where the fun began...

## TODO: add some stuff here about errors and trying to troubleshoot


Digging down into [`terra/RcppFunctions.cpp`](https://github.com/rspatial/terra/blob/master/src/RcppFunctions.cpp#L13-L27) reveals the answer to my issues.

```cpp
#if GDAL_VERSION_MAJOR >= 3
#include "proj.h"
#define projh
#if PROJ_VERSION_MAJOR > 7
# define PROJ_71
#else
# if PROJ_VERSION_MAJOR == 7
#  if PROJ_VERSION_MINOR >= 1
#   define PROJ_71
#  endif
# endif
#endif
#else
#include <proj_api.h>
#endif
```

Because my major version of GDAL is less than 3 this file wades into using the old `proj_api.h` header file and prompting the (quite legitimate) errors i've been having.
So let's update my `gdal` version in my `r-geos` Conda environment and see if everything plays nicely...

```bash
(r-geos)$ conda install gdal=3
```

That installs with some minor tweaks to my environment and now I can test out installing `raster`.

```bash
(r-geos) $ R -e 'install.packages("raster");'
```

And voila! It works!

## Some exciting conclusion