cmake_minimum_required (VERSION 3.5.1)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_COLOR_MAKEFILE   ON)

#SET PROJECT NAME
SET(PROJECT_NAME_ORIG "foo" )

#SET DEPENDENCIES DIRECTORY
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

SET(DEPS_DIR ${MODULES_DIR})
SET(TMP0 ${CMAKE_CURRENT_SOURCE_DIR}/${DEPS_DIR})
GETPROJECTS(DEPS ${TMP0})
message(STATUS "Dependencies found in ${TMP0}: ${DEPS}")
FOREACH(subdir ${DEPS})
    add_subdirectory(${DEPS_DIR}/${subdir})
ENDFOREACH()

##################################
######### BUILD PROJECT ##########
##################################

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
#find_package(catkin REQUIRED COMPONENTS
#  roscpp
#  rospy
#  std_msgs
#  actionlib
#  roslaunch
#  std_srvs
#)

#catkin_package(
#   CATKIN_DEPENDS
#    roscpp
#    rospy
#    std_msgs
#)

#roslaunch_add_file_check(launch)

## Add support for C++11, supported in ROS Kinetic and newer
#add_definitions(-fPIC -std=c++11)

## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)

#SET(SRCPATH src/)
#SET(${PROJECT_NAME}_INCLUDE_DIRS
#    ${SRCPATH}lib/include
#)

#include_directories(
#  ${${PROJECT_NAME}_INCLUDE_DIRS}
#  ${catkin_INCLUDE_DIRS}
#)

#SET(${PROJECT_NAME}_CPP_FILES
#                        ${SRCPATH}lib/src/module1.cpp
#                        ${SRCPATH}lib/src/...
#                        )

#add_library(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_CPP_FILES})
#target_link_libraries(${PROJECT_NAME} ${DEP1_GIT})
#target_include_directories(${PROJECT_NAME} PUBLIC ${${PROJECT_NAME}_INCLUDE_DIRS})


#MODULES_DIR EXAMPLE:
#./gitsubmodules: (NOTE: SET(MODULES_DIR "gitsubmodules" )))
#  dep0/
#    .git
#    CMakeLists.txt
#    ...
#  dep1/
#    .git
#    CMakeLists.txt
#    ...

#add_executable (main ${SRCPATH}main/main.cpp)
#target_link_libraries(${ROVA} ${DEP2_GIT} ... ${catkin_LIBRARIES} ${PROJECT_NAME})


#install(TARGETS ${PROJECT_NAME} main
#        LIBRARY DESTINATION "lib"
#        RUNTIME DESTINATION "bin"
#        )
#install(DIRECTORY "${SRCPATH}lib/include/${PROJECT_NAME}" DESTINATION "include")

message("End ${PROJECT_NAME_ORIG}")
