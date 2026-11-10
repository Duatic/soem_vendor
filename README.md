# soem_vendor

[![Humble](https://github.com/Duatic/soem_vendor/raw/badges/humble.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Jazzy](https://github.com/Duatic/soem_vendor/raw/badges/jazzy.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Kilted](https://github.com/Duatic/soem_vendor/raw/badges/kilted.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Lyrical](https://github.com/Duatic/soem_vendor/raw/badges/lyrical.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Rolling](https://github.com/Duatic/soem_vendor/raw/badges/rolling.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)

The Simple Open EtherCAT Master Library [(SOEM)](https://github.com/OpenEtherCATsociety/SOEM) as a plain C library.

The upstream library is fetched with `FetchContent` from the SOEM repository at the `v1.4.0` commit (`abbf0d4`). The apt package `soem-vendor` installs it in `/usr`, once per Ubuntu release. It also builds in a colcon workspace.

# Usage

```cmake
find_package(soem_vendor REQUIRED)
target_link_libraries(my_target soem_vendor::soem_vendor)
```

```c
#include <soem_vendor/ethercat.h>
```

A ROS package declares `<depend>soem_vendor</depend>` and links the target as above: `ament_target_dependencies` does not link a plain CMake package.

# Upgrading from 0.4

Up to 0.4, this was a ROS package (`ros-<distro>-soem-vendor`, in `/opt/ros`). The names are unchanged, but with ROS sourced the old copy is found first, both by CMake and at runtime. Remove it after upgrading:

```bash
sudo apt autoremove   # or: sudo apt remove 'ros-*-soem-vendor'
```

# Licensing

This package is licensed under BSD 3-Clause [license](./LICENSE).\
SOEM itself is [licensed](https://github.com/OpenEtherCATsociety/SOEM/blob/v1.4.0/LICENSE) under GPLv2 with an exception for static and dynamic linking.
__NOTE:__ This is only true until version `1.4.0` of SOEM. With version `2.0.0` SOEM is now licensed under GPLv3. Therefore this package uses version `1.4.0`

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

Without adding the `soem_vendor` include directory to your CMakeLists the compilation will fail because the headers are not found. With this workaround this step is not needed anymore.
