

############################ PROJECT NAME
# The name of this cmake project
PROJECT( CORPUS )


############################ VERSION
# Check cMake version
CMAKE_MINIMUM_REQUIRED( VERSION 2.2.0 )


############################ COLORIZE MAKEFILE
# Colorized output
SET( CMAKE_COLOR_MAKEFILE TRUE )


############################ NBCOMMON
# Ensure the TRUNK_PATH variable is set

IF ("x$ENV{TRUNK_PATH}x" STREQUAL "xx")
  GET_FILENAME_COMPONENT( TRUNK_PATH ../../ ABSOLUTE)
  SET( ENV{TRUNK_PATH} ${TRUNK_PATH} )
  MESSAGE( STATUS 
    "Environment variable TRUNK_PATH was not set, reseting to default ${TRUNK_PATH}!" )
ELSE ("x$ENV{TRUNK_PATH}x" STREQUAL "xx")
  SET( TRUNK_PATH $ENV{TRUNK_PATH} )
ENDIF ("x$ENV{TRUNK_PATH}x" STREQUAL "xx")


############################ BUILDCONFIG
# Include the config file for this build
INCLUDE( buildconfig.cmake )


############################ BASE DEFINITIONS
# Include  the basic definitions accross all projects
INCLUDE( ${TRUNK_PATH}/cmake/base_definitions.cmake )


############################ DISPLAY
# Display summary of options

MESSAGE( STATUS "" )
MESSAGE( STATUS "...:::: Configuration -  CORPUS ::::..." )
MESSAGE( STATUS "" )


############################ FIND PACKAGE
# Libraries specifics. Define here cmake packages
# usefull to your project

FIND_PACKAGE( NBCOMMON REQUIRED )
FIND_PACKAGE( ALCOMMON REQUIRED )
FIND_PACKAGE( PTHREAD  REQUIRED )
FIND_PACKAGE( PYTHON   REQUIRED )
FIND_PACKAGE( TINYXML  REQUIRED )


############################ CONFIG.H GENERATION
CONFIGURE_FILE(
  ${CORPUS_CMAKE_DIR}/corpusconfig.h.in
  ${CORPUS_INCLUDE_DIR}/corpusconfig.h
  ESCAPE_QUOTES
  )


############################ DEFINITIONS
# Add here definition needed.
ADD_DEFINITIONS( ${NBCOMMON_DEFINITIONS}
                 ${ALCOMMON_DEFINITIONS}
		 ${TINYXML_DEFINITIONS}
                 ${PTHREAD_DEFINITIONS}
                 ${PYTHON_DEFINITIONS}
                 )


############################ INCLUDE DIRECTORY
# Define include directories
INCLUDE_DIRECTORIES( ${NBCOMMON_INCLUDE_DIR}
                     ${ALCOMMON_INCLUDE_DIR}
		     ${TINYXML_INCLUDE_DIR}
                     ${PTHREAD_INCLUDE_DIR}
                     ${PYTHON_INCLUDE_DIR}
                     ${VISION_INCLUDE_DIR}
                     ${CORPUS_INCLUDE_DIR}
                     )


############################ PROJECT SOURCES FILES 
# Add here source files needed to compile this project
SET( SENSORS_SRCS ${CORPUS_INCLUDE_DIR}/Sensors
                  ${CORPUS_INCLUDE_DIR}/NaoPose
                  ${VISION_INCLUDE_DIR}/matrix
                  )
SET( LEDS_SRCS   ${CORPUS_INCLUDE_DIR}/_ledsmodule )

IF( PYTHON_SHARED_CORPUS )
  LIST( APPEND SENSORS_SRCS ${VISION_INCLUDE_DIR}/Pose
                            ${VISION_INCLUDE_DIR}/PoseMatrix
                            )
ENDIF( PYTHON_SHARED_CORPUS )

SET( CORPUS_PYTHON_SRCS ${CORPUS_INCLUDE_DIR}/leds.py
                        ${CORPUS_INCLUDE_DIR}/sensors.py
                        )

SET( CORPUS_TEST_DIR ${CORPUS_INCLUDE_DIR}/test )


############################ CHECK LIBRARY / EXECUTABLE OPTION
# Configure the output execuatable or shared library, and the install
# commands

IF( PYTHON_SHARED_CORPUS )
  ADD_LIBRARY(
    ${SENSORS_TARGET}
    SHARED
    ${SENSORS_SRCS}
    )
  ADD_LIBRARY(
    ${LEDS_TARGET}
    SHARED
    ${LEDS_SRCS}
    )
  GET_TARGET_PROPERTY( SENSORS_TARGET_LOCATION ${SENSORS_TARGET} LOCATION )
  GET_TARGET_PROPERTY( LEDS_TARGET_LOCATION   ${LEDS_TARGET}   LOCATION )
  # Install to TARGETmodule.so, instead of lib_TARGET.so
  INSTALL(
    FILES ${SENSORS_TARGET_LOCATION}
    DESTINATION ${OUTPUT_ROOT_DIR_LIB}
    CONFIGURATIONS "" Debug Release
    RENAME ${SENSORS_TARGET}module.so
    )
  INSTALL(
    FILES ${LEDS_TARGET_LOCATION}
    DESTINATION ${OUTPUT_ROOT_DIR_LIB}
    CONFIGURATIONS "" Debug Release
    RENAME ${LEDS_TARGET}module.so
    )
ELSE( PYTHON_SHARED_CORPUS )
  ADD_LIBRARY(
    ${SENSORS_TARGET}
    STATIC
    ${SENSORS_SRCS}
    )
  ADD_LIBRARY(
    ${LEDS_TARGET}
    STATIC
    ${LEDS_SRCS}
    )
  # No INSTALL target when static
ENDIF( PYTHON_SHARED_CORPUS )

INSTALL(
  FILES ${CORPUS_PYTHON_SRCS}
  DESTINATION ${OUTPUT_ROOT_DIR_LIB}
  CONFIGURATIONS "" Debug Release
  )


############################ LINKER LANGUAGE
# For Mac's, set the target link language to use

#IF( APPLE )
  SET_TARGET_PROPERTIES( ${SENSORS_TARGET} PROPERTIES LINKER_LANGUAGE CXX )
  SET_TARGET_PROPERTIES( ${LEDS_TARGET}   PROPERTIES LINKER_LANGUAGE CXX )
#ENDIF( APPLE )


############################ SET LIBRARIES TO LINK WITH
TARGET_LINK_LIBRARIES( ${SENSORS_TARGET} ${NBCOMMON_LIBRARIES}
                                         ${ALCOMMON_LIBRARIES}
                                         ${TINYXML_LIBRARIES}
                                         ${PTHREAD_LIBRARIES}
                                         ${PYTHON_LIBRARIES}
                                         ${VISION_LIBRARIES}
                                         )
TARGET_LINK_LIBRARIES( ${LEDS_TARGET}   ${NBCOMMON_LIBRARIES}
                                         ${ALCOMMON_LIBRARIES}
                                         ${TINYXML_LIBRARIES}
                                         ${PTHREAD_LIBRARIES}
                                         ${PYTHON_LIBRARIES}
                                         )


############################ (SUB)DIRECTORY COMPILATION
# Set the sudirectories (some may not actually be subdirectories) to
# include in this package
#SUBDIRS(
#  )


