cmake_minimum_required(VERSION 3.7)

project(eigen_vendor)

find_package(ament_cmake REQUIRED)

# Eigen is a header-only linear-algebra library. orocos_kdl (and tf2_eigen,
# many ROS2 math consumers) need it, but lyrical no longer ships an eigen
# colcon package and VxWorks has no system libeigen3-dev. This vendor builds
# Eigen from source (cross-compiled is a no-op for a header-only lib, but we
# install its cmake config so find_package(Eigen3)/eigen3_cmake_module resolve)
# and MUST build before any consumer (orocos_kdl_vendor <depend>s it).
#
# Version 3.4.0 matches the harness pkg/eigen 3PP AND Ubuntu 26.04 libeigen3-dev.

option(FORCE_BUILD_VENDOR_PKG "Build Eigen from source" OFF)

if(NOT FORCE_BUILD_VENDOR_PKG)
  find_package(Eigen3 3.4 QUIET CONFIG)
endif()

macro(build_eigen)
  set(extra_cmake_args)
  if(DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()
  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND extra_cmake_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
  endif()
  # Match the harness pkg/eigen 3PP build options.
  list(APPEND extra_cmake_args
    -DEIGEN_TEST_CXX11=ON
    -DBUILD_TESTING=OFF
    -DEIGEN_BUILD_DOC=OFF)

  include(ExternalProject)

  externalproject_add(eigen_external
    URL https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.tar.bz2
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    TIMEOUT 600
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install
      ${extra_cmake_args}
  )

  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}
    USE_SOURCE_PERMISSIONS
  )
endmacro()

if(NOT Eigen3_FOUND)
  message(STATUS "Did not find Eigen3 3.4, building from source")
  build_eigen()
endif()

ament_package()
