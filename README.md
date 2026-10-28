# soem_vendor

[![Humble](https://github.com/Duatic/soem_vendor/raw/badges/humble.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Jazzy](https://github.com/Duatic/soem_vendor/raw/badges/jazzy.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Kilted](https://github.com/Duatic/soem_vendor/raw/badges/kilted.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Lyrical](https://github.com/Duatic/soem_vendor/raw/badges/lyrical.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)
[![Rolling](https://github.com/Duatic/soem_vendor/raw/badges/rolling.svg)](https://github.com/Duatic/soem_vendor/actions/workflows/ci.yml)

ROS2 packaging for the Simple Open EtherCAT Master Library [(SOEM)](https://github.com/OpenEtherCATsociety/SOEM)


The upstream library is fetched with `FetchContent` from the SOEM repository at the `v1.4.0` commit (`abbf0d4`) and packaged in a ROS compatible way.

# Licensing

This package is licensed under BSD 3-Clause [license](./LICENSE).\
SOEM itself is [licensed](https://github.com/OpenEtherCATsociety/SOEM/blob/v1.4.0/LICENSE) under GPLv2 with an exception for static and dynamic linking.
__NOTE:__ This is only true until version `1.4.0` of SOEM. With version `2.0.0` SOEM is now licensed under GPLv3. Therefore this package uses version `1.4.0`

# EtherCAT

EtherCAT® is a registered trademark and patented technology, licensed by Beckhoff Automation GmbH, Germany.

The EtherCAT Technology, and the trade name “EtherCAT” are the intellectual property of, and protected by Beckhoff Automation GmbH. You may review, edit, compile, link, debug the EtherCAT Master Code, provided that before implementing into a product, using, selling or otherwise distributing an EtherCAT Master License is obtained from Beckhoff Automation GmbH & Co.KG.

To receive a copy of the EtherCAT Master License contact Beckhoff Automation GmbH & Co.KG, Huelshorstweg 20, D-33415 Verl, Germany (www.beckhoff.com).

The package installs this notice as `share/soem_vendor/ETHERCAT_NOTICE`.

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
