cmake_minimum_required(VERSION 2.8.3)
project(ifopt_snopt)
##############
## PREAMBLE ##
##############
add_compile_options(-std=c++11)
find_package(ifopt_core REQUIRED)
if(DEFINED BUILD_ROOT)
  set(GTEST_DIR "${BUILD_ROOT}/third_party/gtest")
else()
  set(GTEST_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/gtest}")
endif()

################
## Find SNOPT ##
################
# adapt this to point to your installed IPOPT folder
set(SNOPT_DIR "/home/winklera/3rd_party_software/snopt_lib")
if(IS_DIRECTORY ${SNOPT_DIR}/include)
  set(SNOPT_INCLUDE_DIRS ${SNOPT_DIR}/include)
  find_library(SNOPT_LIB1 snopt7_cpp ${SNOPT_DIR}/lib)
  find_library(SNOPT_LIB2 snopt7 ${SNOPT_DIR}/lib)
else()
  message(WARNING "SNOPT directory \"" ${SNOPT_DIR} "\" NOT found "
                  "-> Not compiling ifopt_snopt.\n" 
                  "Specify path to your installed SNOPT installation here.")
  return()
endif()

# adapt depending on your SNOPT version
set(SNOPT76 FALSE)
if(${SNOPT76})
  message("SNOPT76 version >= SNOPT 7.6 detected")
endif()

###########
## BUILD ##
###########
if(${SNOPT76})
 add_library(${PROJECT_NAME} src/snopt76_adapter.cc)
else()
 add_library(${PROJECT_NAME} src/snopt_adapter.cc)
endif()
target_include_directories(${PROJECT_NAME} PUBLIC
  include
  ${SNOPT_INCLUDE_DIRS}
  ${ifopt_core_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME} PUBLIC
  ${ifopt_core_LIBRARIES}
  ${SNOPT_LIB1}
  ${SNOPT_LIB2}
)

############
## EXPORT ##
############
# for local build
export(PACKAGE ifopt_snopt)
get_property(ifopt_snopt_INCLUDE_DIRS DIRECTORY 
  ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
get_property(ifopt_snopt_LIBRARIES TARGET ifopt_snopt PROPERTY LIBRARY_OUTPUT_NAME) 
configure_file(ifopt_snoptConfig.cmake.in
  "${PROJECT_BINARY_DIR}/ifopt_snoptConfig.cmake" @ONLY)
# for external build
set(ifopt_snopt_INCLUDE_DIRS ${CMAKE_INSTALL_PREFIX}/include include ${ifopt_core_INCLUDE_DIRS} ${SNOPT_INCLUDE_DIRS})
set(ifopt_snopt_LIBRARIES ${CMAKE_INSTALL_PREFIX}/lib/libifopt_snopt.a ${ifopt_core_LIBRARIES} ${SNOPT_LIB1} ${SNOPT_LIB2})
configure_file(ifopt_snoptConfig.cmake.in
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/ifopt_snoptConfig.cmake" @ONLY)

#############
## INSTALL ##
#############
install(FILES "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/ifopt_snoptConfig.cmake"
  DESTINATION share/ifopt_snopt/cmake COMPONENT dev)
install(TARGETS ifopt_snopt DESTINATION lib)
install(DIRECTORY include/ifopt_snopt DESTINATION include)

#############
## TESTING ##
#############
if(EXISTS ${GTEST_DIR} AND BUILD_TEST)
  add_executable(ifopt_snopt-test
      test/ex_test_snopt.cc
      )
  if(${SNOPT76})
      target_compile_definitions(ifopt_snopt-test PRIVATE SNOPT_ON=1)
  endif()
  target_link_libraries(ifopt_snopt-test ifopt_snopt gtest gtest_main pthread)
  add_test(NAME ifopt_snopt-test COMMAND ifopt_snopt-test)
endif()
