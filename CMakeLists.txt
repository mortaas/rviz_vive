##View RVIZ on the HTC Vive
##Copyright (C) 2018 Andre Gilerson (andre.gilerson@gmail.com)

##Permission is hereby granted, free of charge,
##to any person obtaining a copy of this software
##and associated documentation files (the "Software"),
##to deal in the Software without restriction,
##including without limitation the rights to use, copy,
##modify, merge, publish, distribute, sublicense, and/or
##sell copies of the Software, and to permit persons to
##whom the Software is furnished to do so, subject to
##the following conditions:

##The above copyright notice and this permission notice
##shall be included in all copies or substantial portions of the Software.

##THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
##EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
##OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
##IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
##DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
##OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE
##USE OR OTHER DEALINGS IN THE SOFTWARE.


cmake_minimum_required(VERSION 3.1.0)
project(rviz_vive)

## Compile as C++11, supported in ROS Kinetic and newer
add_compile_options(-std=c++11)

##Hardcoded OpenVR Path please change
# set(OPENVR "~/lib/openvr")
# include_directories("${OPENVR}/headers")
# link_directories("${OPENVR}/lib/linux64")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 ")


###########
## Paths ##
###########

## Set OpenVR library path
## ENV reads paths from a system environment variable
set(OPENVR "$ENV{HOME}/lib/openvr")

set(THIRDPARTY_DIR ${OPENVR}/samples/thirdparty)
set(SHARED_SRC_DIR ${OPENVR}/samples/shared)

## OpenVR SDK path
find_library(OPENVR_LIBRARIES
  NAMES
    ## Specify one or more possible names for the library
    openvr_api
  PATHS
    ## Specify directories to search in addition to the default locations
    ${OPENVR}/bin
    ${OPENVR}/lib
  PATH_SUFFIXES
    ## Specify additional subdirectories to check below each directory location otherwise considered
    linux64
  ## If NO_DEFAULT_PATH is specified, then no additional paths are added to the search
  NO_DEFAULT_PATH
  ## If NO_CMAKE_FIND_ROOT_PATH is used then CMAKE_FIND_ROOT_PATH will not be used
  NO_CMAKE_FIND_ROOT_PATH
)
set(OPENVR_INCLUDE_DIR ${OPENVR}/headers)


find_package(catkin REQUIRED COMPONENTS rviz)
catkin_package()
include_directories(${catkin_INCLUDE_DIRS} ${OPENVR_INCLUDE_DIR})

## This setting causes Qt's "MOC" generation to happen automatically.
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
## This plugin includes Qt widgets, so we must include Qt.
## We'll use the version that rviz used so they are compatible.
if(rviz_QT_VERSION VERSION_LESS "5")
  message(STATUS "Using Qt4 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt4 ${rviz_QT_VERSION} EXACT REQUIRED QtCore QtGui)
  ## pull in all required include dirs, define QT_LIBRARIES, etc.
  include(${QT_USE_FILE})
else()
  message(STATUS "Using Qt5 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt5 ${rviz_QT_VERSION} EXACT REQUIRED Core Widgets)
  ## make target_link_libraries(${QT_LIBRARIES}) pull in all required dependencies
  set(QT_LIBRARIES Qt5::Widgets)
endif()

add_definitions(-DQT_NO_KEYWORDS)

set(SRC_FILES
  src/vive_display.cpp
)

add_library(${PROJECT_NAME} ${SRC_FILES})

target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES} ${catkin_LIBRARIES} ${OPENVR_LIBRARIES} ${SDL2_LIBRARY} ${GLEW_LIBRARIES})

## Install rules

install(TARGETS
  ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(FILES
  plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
