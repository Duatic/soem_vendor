# soem_vendor

[![Rolling Build Main](https://github.com/Duatic/soem_vendor/actions/workflows/build-rolling.yml/badge.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/build-rolling.yml)
[![Jazzy Build Main](https://github.com/Duatic/soem_vendor/actions/workflows/build-jazzy.yml/badge.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/build-jazzy.yml)
[![Humble Build Main](https://github.com/Duatic/soem_vendor/actions/workflows/build-humble.yml/badge.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/build-humble.yml)

ROS2 packaging for the Simple Open EtherCAT Master Library [(SOEM)](https://github.com/OpenEtherCATsociety/SOEM)


The upstream library is imported via `FetchContent` from the SOEM repository and packaged in a ROS compatible way.

# Licensing

This package is licensed under BSD 3-Clause [license](./LICENSE).\
SOEM itself is [licensed](https://github.com/OpenEtherCATsociety/SOEM/blob/master/LICENSE) under GPLv2 with an exception for static and dynamic linking.


# Quirks

In the CMakeList we prefix the include statements for all SOEM headers in the export headers with `soem_vendor`. Otherwise compilation of downstream packages will fail.

__Background:__ The reason this workaround is needed is that SOEM does not package its headers into a project specific subfolder as you'd expect it from most modern cmake and ROS packages. When you access SOEM from your project you would want to write statements such as:

```
#include <soem_vendor/a_soem_header.h>
```

On the contrary the headers themselves directly include each other:

```
#include <a_soem_header.h>
```

Without adding the `soem_vendor` include directory to your CMakeLists the compilation will fail because the headers are not found.


__Recommendation:__ If you write a library do NOT expose SOEM to the outside. In general we recommend the use of the [soem_cpp](https://github.com/duatic/soem_cpp) wrapper for C++ projects.
