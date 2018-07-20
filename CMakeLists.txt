cmake_minimum_required (VERSION 3.1)

project ("Thrift Tutorial")

set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${CMAKE_CURRENT_SOURCE_DIR}/lib/thrift/build/cmake")

# Read the version information from the Autoconf file
file (STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/lib/thrift/configure.ac" CONFIGURE_AC REGEX "AC_INIT\\(.*\\)" )
message(STATUS "Parsed Thrift package version: ${CONFIGURE_AC}")

# The following variable is used in the version.h.in file
string(REGEX REPLACE "AC_INIT\\(\\[.*\\], \\[([0-9]+\\.[0-9]+\\.[0-9]+(-dev)?)\\]\\)" "\\1" PACKAGE_VERSION ${CONFIGURE_AC})
message(STATUS "Parsed Thrift package version: ${PACKAGE_VERSION}")

# These are internal to CMake
string(REGEX REPLACE "([0-9]+\\.[0-9]+\\.[0-9]+)(-dev)?" "\\1" thrift_VERSION ${PACKAGE_VERSION})
string(REGEX REPLACE "([0-9]+)\\.[0-9]+\\.[0-9]+" "\\1" thrift_VERSION_MAJOR ${thrift_VERSION})
string(REGEX REPLACE "[0-9]+\\.([0-9]+)\\.[0-9]+" "\\1" thrift_VERSION_MINOR ${thrift_VERSION})
string(REGEX REPLACE "[0-9]+\\.[0-9]+\\.([0-9]+)" "\\1" thrift_VERSION_PATCH ${thrift_VERSION})
message(STATUS "Parsed Thrift version: ${thrift_VERSION} (${thrift_VERSION_MAJOR}.${thrift_VERSION_MINOR}.${thrift_VERSION_PATCH})")

# Some default settings
include(DefineCMakeDefaults)

# Build time options are defined here
include(DefineOptions)
include(DefineInstallationPaths)

# Based on the options set some platform specifics
include(DefinePlatformSpecifc)

# Generate the config.h file
include(ConfigureChecks)

# Package it
include(CPackConfig)

find_package(Threads)

if(BUILD_COMPILER)
    if(NOT EXISTS ${THRIFT_COMPILER})
        set(THRIFT_COMPILER $<TARGET_FILE:thrift-compiler>)
    endif()
    add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/lib/thrift/compiler/cpp)
elseif(EXISTS ${THRIFT_COMPILER})
    add_executable(thrift-compiler IMPORTED)
    set_property(TARGET thrift-compiler PROPERTY IMPORTED_LOCATION ${THRIFT_COMPILER})
endif()

if(BUILD_CPP)
    add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/lib/thrift/lib/cpp)
    if(BUILD_TUTORIALS)
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)
    endif()
endif()
