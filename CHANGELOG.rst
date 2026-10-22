^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Changelog for package soem_vendor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

0.4.1 (2026-10-22)
------------------
* Pin SOEM to the v1.4.0 commit, and fetch it in Debian package builds too
* Route the package to the public apt archive
* Use SPDX license names, declaring SOEM's linking exception
* Install the license texts with the package
* Publish the package's source beside it in the apt archive
* Add the soem_vendor::soem_vendor alias target
* Export the soem_vendor::soem_vendor target, so downstream packages can link it without ament_target_dependencies

0.4.0 (2026-03-19)
------------------
* Build and install `slave_info` tool

0.3.0 (2025-05-02)
------------------
* Some documentation fixes

0.2.0 (2025-03-01)
------------------
* Setup CI
* Implement a better handling of SOEM headers
* Adapt README

0.1.1 (2025-02-28)
------------------
* Fix the include directory installation paths

0.1.0 (2025-02-28)
-------------------
* Initial release which packages `SOEM 1.4.0 <https://github.com/OpenEtherCATsociety/SOEM/releases/tag/v1.4.0>`_
