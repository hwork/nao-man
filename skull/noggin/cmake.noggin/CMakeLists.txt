##############################
#
# WELCOME 
#
# This is the main cmake configuration file 
# associate to your projects
# You can define here specific compilation options
# 
# More about cmake : www.cmake.org
#
# Using cmake command line : 
# 1. Create a build directory : mkdir build
# 2. Configure cmake project : cd build; cmake ..
#   You can also generate project with your favorite IDE : 
#   Example : Under Mac osx use "cmake .. -G XCode"
#   see cmake help for more information
# 3. Edit project option : ccmake .
# 4. Build it : make 



############################ PROJECT NAME
# The name of this cmake project
PROJECT( NOGGIN )


############################ CMAKE VERSION
# Check cMake version
CMAKE_MINIMUM_REQUIRED( VERSION 2.6.0 )


############################ COLORIZE MAKEFILE
# Colorized output
SET( CMAKE_COLOR_MAKEFILE TRUE )


############################ NBCOMMON
# Ensure the TRUNK_PATH variable is set

IF ("x$ENV{TRUNK_PATH}x" STREQUAL "xx")
  GET_FILENAME_COMPONENT( TRUNK_PATH ../../../ ABSOLUTE)
  SET( ENV{TRUNK_PATH} ${TRUNK_PATH} )
  MESSAGE( STATUS 
    "Environment variable TRUNK_PATH was not set, reseting to default ${TRUNK_PATH}!" )
ELSE ("x$ENV{TRUNK_PATH}x" STREQUAL "xx")
  SET( TRUNK_PATH $ENV{TRUNK_PATH} )
ENDIF ("x$ENV{TRUNK_PATH}x" STREQUAL "xx")


############################ CMAKE PACKAGE DIRECTORY
# Set cmakeModules folder
SET( CMAKE_MODULE_PATH $ENV{TRUNK_PATH}/cmake )


############################ BUILDCONFIG
# Include the config file for this build
INCLUDE( buildconfig.cmake )


############################ BASE DEFINITIONS
# Include  the basic definitions accross all projects
INCLUDE( ${CMAKE_MODULE_PATH}/base_definitions.cmake )


############################ DISPLAY
# Display summary of options

MESSAGE( STATUS "" )
MESSAGE( STATUS "...:::: Configuration -  NOGGIN ::::..." )
MESSAGE( STATUS "" )


############################ FIND PACKAGE
# Libraries specifics. Define here cmake packages
# usefull to your project

FIND_PACKAGE( NBCOMMON REQUIRED )
FIND_PACKAGE( PYTHON   REQUIRED )


############################ CONFIG.H GENERATION
CONFIGURE_FILE(
  "${NOGGIN_CMAKE_DIR}/nogginconfig.in"
  "${NOGGIN_INCLUDE_DIR}/nogginconfig.h"
    ESCAPE_QUOTES
  )
CONFIGURE_FILE(
  "${NOGGIN_INCLUDE_DIR}/noggin/players/Switch.py.in"
  "${NOGGIN_INCLUDE_DIR}/noggin/players/Switch.py"
  )


############################ DEFINITION
# Add here definition needed.
ADD_DEFINITIONS( ${NBCOMMON_DEFINITIONS} ${PYTHON_DEFINITIONS}
                 ${VISION_DEFINITIONS} )


############################ INCLUDE DIRECTORY
# Define include directories
INCLUDE_DIRECTORIES( ${NBCOMMON_INCLUDE_DIR}
                     ${PYTHON_INCLUDE_DIR}
                     ${VISION_INCLUDE_DIR}
                     ${CORPUS_INCLUDE_DIR}
                     ${NOGGIN_INCLUDE_DIR}
                     )


############################ PROJECT SOURCES FILES 
# Add here source files needed to compile this project

SET( NOGGIN_SRCS ${NOGGIN_INCLUDE_DIR}/Noggin.cpp
                 )


SET( NOGGIN_PYTHON_DIRS ${NOGGIN_INCLUDE_DIR}/noggin
                        ${NOGGIN_INCLUDE_DIR}/util )

SET( NOGGIN_EXT_SRCS ${NOGGIN_INCLUDE_DIR}/_ext/comm.py
                     ${NOGGIN_INCLUDE_DIR}/_ext/naovision.py
                     ${NOGGIN_INCLUDE_DIR}/_ext/potentialfield.py
                     )

SET( NOGGIN_TEST_DIR ${NOGGIN_INCLUDE_DIR}/test )


############################ BUILD TARGETS

ADD_LIBRARY(
  ${NOGGIN_TARGET}
  STATIC
  ${NOGGIN_SRCS}
  )
INSTALL(
  TARGETS ${NOGGIN_TARGET}
  ARCHIVE DESTINATION ${OUTPUT_ROOT_DIR_LIB}
  CONFIGURATIONS Debug
  )
INSTALL(
  DIRECTORY ${NOGGIN_PYTHON_DIRS}
  DESTINATION ${OUTPUT_ROOT_DIR_LIB}
  CONFIGURATIONS "" Debug Release
  PATTERN .svn EXCLUDE
  )
INSTALL(
  FILES ${NOGGIN_EXT_SRCS}
  DESTINATION ${OUTPUT_ROOT_DIR_LIB}
  CONFIGURATIONS Debug
  )
INSTALL(
  FILES ${NOGGIN_TEST_DIR}/test_noggin.py
        ${NOGGIN_TEST_DIR}/test_vision.py
  DESTINATION ${OUTPUT_ROOT_DIR_LIB}
  CONFIGURATIONS Debug
  )


############################ LINKER LANGUAGE
# For Mac's, set the target link language to use

#IF( APPLE )
  SET_TARGET_PROPERTIES(
    ${NOGGIN_TARGET} 
    PROPERTIES LINKER_LANGUAGE CXX
    )
#ENDIF( APPLE )


############################ SET LIBRARIES TO LINK WITH
TARGET_LINK_LIBRARIES(
  ${NOGGIN_TARGET}
  ${NBCOMMON_LIBRARIES}
  ${PYTHON_LIBRARIES}
  ${VISION_TARGET}
  ${LEDS_TARGET}
  ${SENSORS_TARGET}
  )


############################ (SUB)DIRECTORY COMPILATION
# Set the sudirectories (some may not actually be subdirectories) to
# include in this package
#SUBDIRS(
#  ${VISION_CMAKE_DIR}
#  )

