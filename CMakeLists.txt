cmake_minimum_required(VERSION 3.15)
project(duatic_soem_vendor C)

include(GNUInstallDirs)


# Fetch the specified version of SOEM
include(FetchContent)
# Debian package builds disconnect FetchContent; SOEM is fetched at its pinned commit regardless
set(FETCHCONTENT_FULLY_DISCONNECTED OFF)
FetchContent_Declare(
    soem
    GIT_REPOSITORY https://github.com/OpenEtherCATsociety/SOEM
    GIT_TAG        abbf0d42e38d6cfbaa4c1e9e8e07ace651c386fd  # v1.4.0
)

FetchContent_GetProperties(soem)
# Make sure that we only download it once
if(NOT soem_POPULATED)
    FetchContent_Populate(soem)
endif()


# Do the handling for different architectures
# See upstream CMakeLists for more information: https://github.com/OpenEtherCATsociety/SOEM/blob/master/CMakeLists.txt
# This has actually some targets removed that are not supported with ROS2
if(WIN32)
  set(OS "win32")
  include_directories(oshw/win32/wpcap/Include)
  if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    link_directories(${CMAKE_CURRENT_LIST_DIR}/oshw/win32/wpcap/Lib/x64)
  elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
    link_directories(${CMAKE_CURRENT_LIST_DIR}/oshw/win32/wpcap/Lib)
  endif()
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /D _CRT_SECURE_NO_WARNINGS")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  /WX")
  set(OS_LIBS wpcap.lib Packet.lib Ws2_32.lib Winmm.lib)
elseif(UNIX AND NOT APPLE)
  set(OS "linux")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
  set(OS_LIBS pthread rt)
elseif(APPLE)
  # This must come *before* linux or MacOSX will identify as Unix.
  set(OS "macosx")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
  set(OS_LIBS pthread pcap)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "rt-kernel")
  set(OS "rtk")
  message("ARCH is ${ARCH}")
  message("BSP is ${BSP}")
  include_directories(oshw/${OS}/${ARCH})
  file(GLOB OSHW_EXTRA_SOURCES oshw/${OS}/${ARCH}/*.c)
  set(OSHW_SOURCES "${OS_HW_SOURCES} ${OSHW_ARCHSOURCES}")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-but-set-variable")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-function")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-format")
  set(OS_LIBS "-Wl,--start-group -l${BSP} -l${ARCH} -lkern -ldev -lsio -lblock -lfs -lusb -llwip -leth -li2c -lrtc -lcan -lnand -lspi -lnor -lpwm -ladc -ltrace -lc -lm -Wl,--end-group")
endif()


# Obtain all relevant source files
# Actually using GLOB here is ok as we do not actually work with the code
file(GLOB SOEM_SOURCES ${soem_SOURCE_DIR}/soem/*.c)
file(GLOB OSAL_SOURCES ${soem_SOURCE_DIR}/osal/${OS}/*.c)
file(GLOB OSHW_SOURCES ${soem_SOURCE_DIR}/oshw/${OS}/*.c)


add_library(duatic_soem_vendor SHARED
    ${SOEM_SOURCES}
    ${OSAL_SOURCES}
    ${OSHW_SOURCES}
    ${OSHW_EXTRA_SOURCES}
)
add_library(duatic_soem_vendor::duatic_soem_vendor ALIAS duatic_soem_vendor)



set_target_properties(duatic_soem_vendor PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_include_directories(duatic_soem_vendor PUBLIC
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/soem>
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/osal>
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/osal/${OS}>
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/oshw/${OS}>
  # Only a single INSTALL_INTERFACE entry as all files are put into the same directory of the vendor package
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)




target_link_libraries(duatic_soem_vendor
    ${OS_LIBS}
)



install(TARGETS duatic_soem_vendor
        EXPORT duatic_soem_vendorTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# find_package(duatic_soem_vendor) provides duatic_soem_vendor::duatic_soem_vendor
install(
  EXPORT duatic_soem_vendorTargets
  FILE duatic_soem_vendorConfig.cmake
  NAMESPACE duatic_soem_vendor::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

# Collect all the header files for installation
file(GLOB SOEM_HEADERS ${soem_SOURCE_DIR}/soem/*.h)
file(GLOB OSAL_HEADERS ${soem_SOURCE_DIR}/osal/osal.h ${soem_SOURCE_DIR}/osal/${OS}/*.h)
file(GLOB OSHW_HEADERS ${soem_SOURCE_DIR}/oshw/${OS}/*.h)


# This is kinda nasty and someone with better CMake/Regex skills could probably write this way better but
# we need to adapt the includes in the exported headers so that another project depending on this vendor package will actually compile
# The reason we need to do this is that SOEM does not have a "project" folder for its includes but always just writes
# #include "some_header.h" on the other hand we want to use it as #include "duatic_soem_vendor/some_header.h".
# So what we do is just adapt all include paths in the exported headers...
set(FIXED_INCLUDES)
foreach(FILE ${SOEM_HEADERS} ${OSAL_HEADERS} ${OSHW_HEADERS})
    get_filename_component(FILE_NAME ${FILE} NAME)
    file(READ ${FILE} CONTENT)
    string(REPLACE "\"ethercattype.h\"" "\"duatic_soem_vendor/ethercattype.h\""  CONTENT "${CONTENT}")
    string(REPLACE "<ethercattype.h>" "\"duatic_soem_vendor/ethercattype.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"nicdrv.h\"" "\"duatic_soem_vendor/nicdrv.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatbase.h\"" "\"duatic_soem_vendor/ethercatbase.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatmain.h\"" "\"duatic_soem_vendor/ethercatmain.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatdc.h\"" "\"duatic_soem_vendor/ethercatdc.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatfoe.h\"" "\"duatic_soem_vendor/ethercatfoe.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatcoe.h\"" "\"duatic_soem_vendor/ethercatcoe.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatsoe.h\"" "\"duatic_soem_vendor/ethercatsoe.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercateoe.h\"" "\"duatic_soem_vendor/ethercateoe.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatconfig.h\"" "\"duatic_soem_vendor/ethercatconfig.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"ethercatprint.h\"" "\"duatic_soem_vendor/ethercatprint.h\""  CONTENT "${CONTENT}")

    string(REPLACE "\"osal.h\"" "\"duatic_soem_vendor/osal.h\""  CONTENT "${CONTENT}")
    string(REPLACE "\"osal_defs.h\"" "\"duatic_soem_vendor/osal_defs.h\""  CONTENT "${CONTENT}")


    file(WRITE "${CMAKE_BINARY_DIR}/fixed_includes/${FILE_NAME}" "${CONTENT}")
    list(APPEND FIXED_INCLUDES "${CMAKE_BINARY_DIR}/fixed_includes/${FILE_NAME}")
endforeach()


# The reason we put all of these headers into the main folder is because its done the same way in SOEM itself
install(FILES ${FIXED_INCLUDES}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

# Build the slave info tool which can be very useful for debugging
add_executable(slave_info ${soem_SOURCE_DIR}/test/linux/slaveinfo/slaveinfo.c)
target_include_directories(slave_info PUBLIC
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/soem>
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/osal>
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/osal/${OS}>
  $<BUILD_INTERFACE:${soem_SOURCE_DIR}/oshw/${OS}>
  # Only a single INSTALL_INTERFACE entry as all files are put into the same directory of the vendor package
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
target_link_libraries(slave_info duatic_soem_vendor)
install(TARGETS slave_info
   DESTINATION ${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME})

# Binary packages carry the license texts, SOEM's included
install(FILES LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME})
install(FILES ${soem_SOURCE_DIR}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME} RENAME LICENSE.SOEM)
