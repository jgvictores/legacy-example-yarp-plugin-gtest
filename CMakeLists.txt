# Copyright: Universidad Carlos III de Madrid (C) 2016;
# Authors: jgvictores
# CopyPolicy: Released under the terms of the GNU GPL v2.0.

# Exploit new cmake 2.6 features (export). 

# reduce warning level with cmake 2.6
cmake_minimum_required(VERSION 2.6)
#cmake policies
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

project(EXAMPLE_YARP_PLUGIN_GTEST)

option(BUILD_SHARED_LIBS "All shared libs" TRUE)

### options: cpp libraries
option(ENABLE_GtestYarp_ExampleLibrary "Choose if you want to compile ExampleLibrary" TRUE)

### options: cpp programs
option(ENABLE_exampleProgram "Choose if you want to compile exampleProgram" TRUE)

### options: force default
option(ENABLE_exampleExtraOption "Enable/disable option exampleExtraOption" TRUE)

if(MSVC)
  MESSAGE(STATUS "Running on windows")    
  set(CMAKE_DEBUG_POSTFIX "d")
endif(MSVC)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING 
    "Choose the type of build, recommanded options are: Debug or Release")
endif(NOT CMAKE_BUILD_TYPE)
# Hide variable to MSVC users, since it is not needed
if (MSVC)
   mark_as_advanced(CMAKE_BUILD_TYPE)
endif(MSVC)

##### test and coverage options
option(ENABLE_tests "Choose if you want to compile tests" TRUE)

######################

### this makes everything go in $EXAMPLE_YARP_PLUGIN_GTEST_DIR/lib and $EXAMPLE_YARP_PLUGIN_GTEST_DIR/bin
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
message(STATUS "Libraries go to ${LIBRARY_OUTPUT_PATH}")
message(STATUS "Executables go to ${EXECUTABLE_OUTPUT_PATH}")
# this doesn't happen automatically for makefiles
make_directory(${LIBRARY_OUTPUT_PATH})
make_directory(${EXECUTABLE_OUTPUT_PATH})
# and let us clean their contents on a "make clean"
##set_directory_properties(PROPERTIES LIBRARY_OUTPUT_PATH ADDITIONAL_MAKE_CLEAN_FILES)
##set_directory_properties(PROPERTIES EXECUTABLE_OUTPUT_PATH ADDITIONAL_MAKE_CLEAN_FILES)
mark_as_advanced(LIBRARY_OUTPUT_PATH EXECUTABLE_OUTPUT_PATH)
mark_as_advanced(CMAKE_BACKWARDS_COMPATIBILITY)
##########################################

# Pick up our cmake modules - they are all in the cmake subdirectory
set(EXAMPLE_YARP_PLUGIN_GTEST_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
# let cmake use them
list(APPEND CMAKE_MODULE_PATH ${EXAMPLE_YARP_PLUGIN_GTEST_MODULE_PATH})

set(EXAMPLE_YARP_PLUGIN_GTEST_INCLUDE_DIRS CACHE INTERNAL "appended header dirs" FORCE)
set(EXAMPLE_YARP_PLUGIN_GTEST_LINK_DIRS CACHE INTERNAL "appended link dirs" FORCE)
set(EXAMPLE_YARP_PLUGIN_GTEST_LIBRARIES CACHE INTERNAL "appended libraries" FORCE)

# add main contents
add_subdirectory(libraries)
add_subdirectory(programs)
if (ENABLE_tests)
    add_subdirectory(test)
endif (ENABLE_tests)

# export our variables to a EXAMPLE_YARP_PLUGIN_GTESTConfig.cmake creation
set(EXAMPLE_YARP_PLUGIN_GTEST_LINK_DIRS ${EXAMPLE_YARP_PLUGIN_GTEST_LINK_DIRS} ${LIBRARY_OUTPUT_PATH})
configure_file(${CMAKE_SOURCE_DIR}/cmake/template/EXAMPLE_YARP_PLUGIN_GTESTConfig.cmake.in
               ${CMAKE_BINARY_DIR}/EXAMPLE_YARP_PLUGIN_GTESTConfig.cmake @ONLY)

# create a symbolic link to the shared directory (as contains models and may be heavy)
# If you want a deep copy, you can use the copy_directory command instead of create_symlink.
#add_custom_target(install_applications COMMAND ${CMAKE_COMMAND} -E create_symlink ${CMAKE_SOURCE_DIR}/share ${CMAKE_SOURCE_DIR}/../share)

#endif(MSVC)
##add_custom_target(nuke
##  "${CMAKE_COMMAND}" -E "remove_directory" "${CMAKE_SOURCE_DIR}/bin" ";"
##  "${CMAKE_COMMAND}" -E "remove_directory" "${CMAKE_SOURCE_DIR}/lib")

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/template/EXAMPLE_YARP_PLUGIN_GTESTConfigUninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/EXAMPLE_YARP_PLUGIN_GTESTConfigUninstall.cmake" IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/EXAMPLE_YARP_PLUGIN_GTESTConfigUninstall.cmake)

