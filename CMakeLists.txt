cmake_minimum_required(VERSION 2.8.3)
project(ifopt)

##############
## PREAMBLE ##
##############
# external package dependencies
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})
# set cache variables
set(BUILD_ROOT ${CMAKE_CURRENT_SOURCE_DIR} CACHE STRING "top-level cmake location" FORCE)
if(NOT BUILD_TEST)
    set(BUILD_TEST false CACHE BOOL "Build unit tests" FORCE)
    set_property(CACHE BUILD_TEST PROPERTY STRINGS "True" "False")
endif()
# build googletest if directory exists and BUILD_TEST is set to True
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/third_party/gtest/" AND BUILD_TEST)
    option(BUILD_GTEST "build gtest" ON)
    add_subdirectory(third_party/gtest third_party/gtest)
    enable_testing()
    include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
endif()

###########
## BUILD ##
###########
add_subdirectory(ifopt_core ifopt_core)
add_subdirectory(ifopt_ipopt ifopt_ipopt)
add_subdirectory(ifopt_snopt ifopt_snopt)

###############
## UNINSTALL ##
###############
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)
add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
