cmake_minimum_required (VERSION 3.5.1)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_COLOR_MAKEFILE   ON)
#It seams that from Ubuntu 17.10 the linker flag --disable-new-dtags is needed
#for the executable to be able to find absolute linked shared libraries with rpath.
#https://stackoverflow.com/questions/47117443/dynamic-linking-with-rpath-not-working-under-ubuntu-17-10
SET(CMAKE_EXE_LINKER_FLAGS "-Wl,--disable-new-dtags")

#SET PROJECT NAME
#NOTE: replace "foo" by your project name
SET(PROJECT_NAME_ORIG "foo" )

#SET DEPENDENCIES DIRECTORY
#NOTE: replace "gitsubmodules" by the directory name where your
#      cmake based git submodules are"
SET(MODULES_DIR "gitsubmodules" )

message("Building ${PROJECT_NAME_ORIG}...")

#BEGIN RESOLVE GIT REVISION
#https://bravenewmethod.com/2017/07/14/git-revision-as-compiler-definition-in-build-with-cmake/
message(STATUS "Resolving GIT Version...")
set(_build_version "unknown")
find_package(Git)
if(GIT_FOUND)
  execute_process(
    COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    OUTPUT_VARIABLE _build_version
    ERROR_QUIET
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  message( STATUS "GIT hash: ${_build_version}")
else()
  message(STATUS "GIT not found")
endif()
string(TIMESTAMP _time_stamp)
#END RESOLVE GIT REVISION

#http://stackoverflow.com/questions/7787823/cmake-how-to-get-the-name-of-all-subdirectories-of-a-directory
function(GETPROJECTS result curdir)
  FILE(GLOB children RELATIVE ${curdir} ${curdir}/*/CMakeLists.txt)
  SET(dirlist "")
  FOREACH(file ${children})
        get_filename_component(subdir ${file} DIRECTORY)
        LIST(APPEND dirlist ${subdir})
  ENDFOREACH()
  SET(${result} ${dirlist} PARENT_SCOPE)
endfunction()

#DECLARE PROJECT
SET(PROJECT_NAME_GIT "${PROJECT_NAME_ORIG}_${_build_version}")
string(TOUPPER ${PROJECT_NAME_ORIG} TMP0)
SET(${TMP0}_GIT "${PROJECT_NAME_GIT}" PARENT_SCOPE)

if (TARGET ${PROJECT_NAME_GIT})
    message(WARNING "${PROJECT_NAME_GIT} has already been built!")
    return()
endif()

project (${PROJECT_NAME_GIT} CXX)
message(STATUS "PROJECT NAME: ${PROJECT_NAME}")

##################################
####### BUILD DEPENDENCIES #######
##################################

### EXAMPLE:
#./gitsubmodules: (NOTE: IF SET(MODULES_DIR "gitsubmodules" )))
#  module0/
#    .git
#    CMakeLists.txt:
#          ...
#          SET(PROJECT_NAME_ORIG "dep0" )
#          ...
#    ...
#  module1/
#    .git
#    CMakeLists.txt:
#           ...
#           SET(PROJECT_NAME_ORIG "dep1" )
#           ...
#    ...

SET(DEPS_DIR ${MODULES_DIR})
SET(TMP0 ${CMAKE_CURRENT_SOURCE_DIR}/${DEPS_DIR})
GETPROJECTS(DEPS ${TMP0})
message(STATUS "Dependencies found in ${TMP0}: ${DEPS}")
FOREACH(subdir ${DEPS})
if(GIT_FOUND)
  SET(SUBPROJECT_PATH ${DEPS_DIR}/${subdir})
  SET(SUBPROJECT_FULL_PATH ${CMAKE_CURRENT_SOURCE_DIR}/${SUBPROJECT_PATH})
  execute_process(
    COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
    WORKING_DIRECTORY ${SUBPROJECT_FULL_PATH}
    OUTPUT_VARIABLE _sub_build_version
    ERROR_QUIET
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  SET(SUBPROJECT_GIT_NAME ${subdir}_${_sub_build_version})
  message(STATUS "Subproject generated git name: " ${SUBPROJECT_GIT_NAME})
  if (TARGET ${subdir})
      message(WARNING "Subproject ${SUBPROJECT_GIT_NAME} (${subdir}) has already been built!")
  else()
      add_subdirectory(${SUBPROJECT_PATH})
  endif()
else()
  message(STATUS "GIT not found")
endif()
ENDFOREACH()

##################################
######### BUILD PROJECT ##########
##################################

## Add support for C++11, supported in ROS Kinetic and newer
#add_definitions(-fPIC -std=c++11)

#SET(SRCPATH src/)
#SET(${PROJECT_NAME}_INCLUDE_DIRS
#    ${CMAKE_CURRENT_SOURCE_DIR}/${SRCPATH}include
#    ${${DEP0_GIT}_INCLUDE_DIRS}
#    ${${DEP1_GIT}_INCLUDE_DIRS}
#)

#include_directories(
#  ${${PROJECT_NAME}_INCLUDE_DIRS}
#)

#SET(${PROJECT_NAME}_CPP_FILES
#                        ${SRCPATH}module0.cpp
#                        ${SRCPATH}module1.cpp
#                            ...
#                        )

#add_library(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_CPP_FILES})
#target_link_libraries(${PROJECT_NAME} ${DEP0_GIT} ${DEP1_GIT})

#add_executable (main ${SRCPATH}main/main.cpp)
#target_link_libraries(main ${LIB2_GIT} ... ${PROJECT_NAME})

#SET(${PROJECT_NAME}_INCLUDE_DIRS ${${PROJECT_NAME}_INCLUDE_DIRS} CACHE STRING "${PROJECT_NAME} include dirs")
#SET(${PROJECT_NAME}_LIBRARIES ${PROJECT_NAME} ${${DEP0_GIT}_LIBRARIES} ${${DEP1_GIT}_LIBRARIES} CACHE STRING "${PROJECT_NAME} libraries")

#install(TARGETS ${PROJECT_NAME} main
#        LIBRARY DESTINATION "lib"
#        RUNTIME DESTINATION "bin"
#        )
#install(DIRECTORY "${SRCPATH}include/${PROJECT_NAME}" DESTINATION "include")

message("End ${PROJECT_NAME_ORIG}")
