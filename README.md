# soem_vendor

[![Humble](https://github.com/Duatic/soem_vendor/raw/badges/humble.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Jazzy](https://github.com/Duatic/soem_vendor/raw/badges/jazzy.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Kilted](https://github.com/Duatic/soem_vendor/raw/badges/kilted.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Lyrical](https://github.com/Duatic/soem_vendor/raw/badges/lyrical.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Rolling](https://github.com/Duatic/soem_vendor/raw/badges/rolling.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)

ROS2 packaging for the Simple Open EtherCAT Master Library [(SOEM)](https://github.com/OpenEtherCATsociety/SOEM)


The upstream library is vendored in `third_party/SOEM` and packaged in a ROS compatible way, so building needs no network.

`third_party/SOEM` is SOEM [v1.4.0](https://github.com/OpenEtherCATsociety/SOEM/tree/v1.4.0) (commit `abbf0d42e38d6cfbaa4c1e9e8e07ace651c386fd`), unmodified except that the prebuilt Windows libraries in `oshw/win32/wpcap/Lib` are left out. To check it against upstream:

```
git clone --depth 1 --branch v1.4.0 https://github.com/OpenEtherCATsociety/SOEM.git /tmp/soem
diff -r --exclude .git --exclude Lib /tmp/soem third_party/SOEM
```

# Licensing

This package is licensed under BSD 3-Clause [license](./LICENSE).\
SOEM itself is [licensed](./third_party/SOEM/LICENSE) under GPLv2 with an exception for static and dynamic linking.
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

Without adding the `soem_vendor` include directory to your CMakeLists the compilation will fail because the headers are not found. With this workaround this step is not needed anymore and allows you to write a default ROS2 CMakeLists.txt.
