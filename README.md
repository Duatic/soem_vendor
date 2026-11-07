# duatic_soem_vendor

[![Humble](https://github.com/Duatic/soem_vendor/raw/badges/humble.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Jazzy](https://github.com/Duatic/soem_vendor/raw/badges/jazzy.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Kilted](https://github.com/Duatic/soem_vendor/raw/badges/kilted.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Lyrical](https://github.com/Duatic/soem_vendor/raw/badges/lyrical.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Rolling](https://github.com/Duatic/soem_vendor/raw/badges/rolling.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)

The Simple Open EtherCAT Master Library [(SOEM)](https://github.com/OpenEtherCATsociety/SOEM) as a plain C library, without ROS.

The upstream library is fetched with `FetchContent` from the SOEM repository at the `v1.4.0` commit (`abbf0d4`). The apt package `duatic-soem-vendor` installs it in `/usr`, once per Ubuntu release. It also builds in a colcon workspace.

# Usage

```cmake
find_package(duatic_soem_vendor REQUIRED)
target_link_libraries(my_target duatic_soem_vendor::duatic_soem_vendor)
```

```c
#include <duatic_soem_vendor/ethercat.h>
```

A ROS package declares `<depend>duatic_soem_vendor</depend>`.

# Migrating from soem_vendor

Up to 0.4, this was the ROS package `soem_vendor` (`ros-<distro>-soem-vendor`, in `/opt/ros`). A dependent replaces `soem_vendor` with `duatic_soem_vendor` in its `package.xml`, its `find_package()`, its target and its includes. On a machine that has the old package installed, remove it after upgrading, so it does not shadow the new one:

```bash
sudo apt remove 'ros-*-soem-vendor'
```

# Licensing

This package is licensed under BSD 3-Clause [license](./LICENSE).\
SOEM itself is [licensed](https://github.com/OpenEtherCATsociety/SOEM/blob/v1.4.0/LICENSE) under GPLv2 with an exception for static and dynamic linking.
__NOTE:__ This is only true until version `1.4.0` of SOEM. With version `2.0.0` SOEM is now licensed under GPLv3. Therefore this package uses version `1.4.0`

# Quirks

In the CMakeList we prefix the include statements for all SOEM headers in the export headers with `duatic_soem_vendor`. Otherwise compilation of downstream packages will fail.

__Background:__ The reason this workaround is needed is that SOEM does not package its headers into a project specific subfolder as you'd expect it from most modern cmake and ROS packages. When you access SOEM from your project you would want to write statements such as:

```
#include <duatic_soem_vendor/a_soem_header.h>
```

On the contrary the headers themselves directly include each other:

```
#include <a_soem_header.h>
```

Without adding the `duatic_soem_vendor` include directory to your CMakeLists the compilation will fail because the headers are not found. With this workaround this step is not needed anymore.
