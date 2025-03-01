# soem_vendor

ROS2 packaging for the Simple Open EtherCAT Master Library [(SOEM)](https://github.com/OpenEtherCATsociety/SOEM)


The upstream library is imported via `FetchContent` from the SOEM repository and packaged in a ROS compatible way. 

# Licensing

This package is licensed under BSD 3-Clause [license](./LICENSE).\
SOEM itself is [licensed](https://github.com/OpenEtherCATsociety/SOEM/blob/master/LICENSE) under GPLv2 with an exception for static and dynamic linking.


# Usage notes

The usage in general is rather straight forward but has a few quirks due to the way SOEM handles headers.

1. Add the dependency to soem_vendor into your `package.xml` and `CMakeLists.txt` as usual
2. For your cmake target that depends on SOEM add:

```cmake
# If necessary replace ${PROJECT_NAME} with your target name
target_include_directories(${PROJECT_NAME} PRIVATE
    ${soem_vendor_INCLUDE_DIRS}/soem_vendor)
```

You __want__ to keep this statement `PRIVATE` to your target as otherwise you will polute all targets that depend on your library with the global SOEM include directory. This can (and will) lead to issues in case a large project depends on different versions of SOEM.


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