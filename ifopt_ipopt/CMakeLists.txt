cmake_minimum_required(VERSION 2.8.3)
project(ifopt_ipopt)

##############
## PREAMBLE ##
##############
add_compile_options(-std=c++11)
find_package(ifopt_core REQUIRED)

################
## Find IPOPT ##
################
# adapt this to point to your installed IPOPT folder
set(IPOPT_DIR "/usr/lib")
if(IS_DIRECTORY ${IPOPT_DIR}/../include/coin)
  set(IPOPT_INCLUDE_DIRS ${IPOPT_DIR}/../include/coin)
  set(IPOPT_LIBRARIES    ${IPOPT_DIR}/libipopt.so)
else()
  message(WARNING "IPOPT directory \"" ${IPOPT_DIR} "\" NOT found " 
      "-> Not compiling ifopt_ipopt. \n" 
      "Specify path to your installed IPOPT installation here.")
  return()
endif()

###########
## BUILD ##
###########
# The IPOPT solver interface
add_library(ifopt_ipopt STATIC src/ipopt_adapter.cc)
target_include_directories(ifopt_ipopt PUBLIC
    include
    ${IPOPT_INCLUDE_DIRS}
    ${ifopt_core_INCLUDE_DIRS}
)
target_link_libraries(ifopt_ipopt PUBLIC
  ${ifopt_core_LIBRARIES}
  ${IPOPT_LIBRARIES}
)
# needed when ipopt installed via apt on ubuntu
target_compile_definitions(ifopt_ipopt PRIVATE HAVE_CSTDDEF)

############
## EXPORT ##
############
# for local build
export(PACKAGE ifopt_ipopt)
get_property(ifopt_ipopt_INCLUDE_DIRS DIRECTORY 
  ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
get_property(ifopt_ipopt_LIBRARIES TARGET ifopt_ipopt PROPERTY LIBRARY_OUTPUT_NAME) 
configure_file(ifopt_ipoptConfig.cmake.in
  "${PROJECT_BINARY_DIR}/ifopt_ipoptConfig.cmake" @ONLY)
# for external build
set(ifopt_ipopt_INCLUDE_DIRS ${CMAKE_INSTALL_PREFIX}/include include ${ifopt_core_INCLUDE_DIRS})
set(ifopt_ipopt_LIBRARIES ${CMAKE_INSTALL_PREFIX}/lib/libifopt_ipopt.a ${ifopt_core_LIBRARIES})
configure_file(ifopt_ipoptConfig.cmake.in
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/ifopt_ipoptConfig.cmake" @ONLY)

#############
## INSTALL ##
#############
install(FILES "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/ifopt_ipoptConfig.cmake"
  DESTINATION share/ifopt_ipopt/cmake COMPONENT dev)
install(TARGETS ifopt_ipopt DESTINATION lib)
install(DIRECTORY include/ifopt_ipopt DESTINATION include)

#############
## TESTING ##
#############
if(DEFINED BUILD_ROOT)
  set(GTEST_DIR "${BUILD_ROOT}/third_party/gtest")
else()
  set(GTEST_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/gtest}")
endif()
if(EXISTS ${GTEST_DIR} AND BUILD_TEST)
  add_executable(ifopt_ipopt-test
      test/ex_test_ipopt.cc
      )
  target_compile_definitions(ifopt_ipopt-test PRIVATE HAVE_CSTDDEF)
  target_link_libraries(ifopt_ipopt-test ifopt_ipopt gtest gtest_main pthread)
  add_test(NAME ifopt_ipopt-test COMMAND ifopt_ipopt-test)
endif()
