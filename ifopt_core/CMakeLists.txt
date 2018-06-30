cmake_minimum_required(VERSION 2.8.3)
project(ifopt_core)

##############
## PREAMBLE ##
##############
# use the c++11 standard
add_compile_options(-std=c++11)
# external package dependencies
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})
# googletest library location
if(DEFINED BUILD_ROOT)
    set(GTEST_DIR "${BUILD_ROOT}/third_party/gtest")
else()
    set(GTEST_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/gtest}")
endif()

###########
## BUILD ##
###########
add_library(ifopt_core STATIC
    src/problem.cc
    src/composite.cc
    src/leaves.cc
)
target_include_directories(ifopt_core PUBLIC include ${EIGEN3_INCLUDE_DIR})
set_target_properties(ifopt_core PROPERTIES 
    LINK_INTERFACE_LIBRARIES ${PROJECT_BINARY_DIR}/libifopt_core.a
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include)

############
## EXPORT ##
############
# for local build
export(PACKAGE ifopt_core)
get_property(ifopt_core_INCLUDE_DIRS DIRECTORY 
    ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
get_property(ifopt_core_LIBRARIES TARGET ifopt_core PROPERTY LINK_INTERFACE_LIBRARIES) 
configure_file(ifopt_coreConfig.cmake.in
    "${PROJECT_BINARY_DIR}/ifopt_coreConfig.cmake" @ONLY)
# for external build
set(ifopt_core_INCLUDE_DIRS ${CMAKE_INSTALL_PREFIX}/include ${EIGEN3_INCLUDE_DIR})
set(ifopt_core_LIBRARIES ${CMAKE_INSTALL_PREFIX}/lib/libifopt_core.a)
configure_file(ifopt_coreConfig.cmake.in
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/ifopt_coreConfig.cmake" @ONLY)

#############
## INSTALL ##
#############
install(FILES "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/ifopt_coreConfig.cmake"
    DESTINATION share/ifopt_core/cmake COMPONENT dev)
install(TARGETS ifopt_core DESTINATION lib)

#############
## TESTING ##
#############
if(EXISTS ${GTEST_DIR} AND BUILD_TEST)
  add_executable(ifopt_core-test 
    test/gtest_main.cc
    test/composite_test.cc
    test/problem_test.cc
    )
  target_link_libraries(ifopt_core-test ifopt_core gtest gtest_main pthread)
  add_test(NAME ifopt_core-test COMMAND ifopt_core-test)
endif()
