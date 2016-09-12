project(OvrvisionPro)
cmake_minimum_required(VERSION 3.0)

OPTION(BUILD_SHARED_LIBS "Build shared libraries" ON)

# --------------------------------------------------------------------------
# Ensure all dependencies are available
FIND_PACKAGE(OpenCV 3.1.0 REQUIRED NO_MODULE)
FIND_PACKAGE(OpenCL REQUIRED)
FIND_PACKAGE(OpenGL REQUIRED)

# --------------------------------------------------------------------------
# Hack section

# Hack to handle #pragma comment(lib, "xyz") in ovrvision_tracking.cpp
LINK_DIRECTORIES(${OpenCV_DIR}/3rdparty/lib/)
# We expect a superbuild project to pass us in the directory that contains
#   ippicvmt.lib
LINK_DIRECTORIES(${PragmaHack_DIR})
# We need to link the directory that contains 
#   opencv_highgui300d.lib (grrrr, version locked!!!)
LINK_DIRECTORIES(${OpenCV_DIR}/lib/)

# Since we're hacking things... let's hack this REEAAALLLLLL good
IF(EXISTS ${OpenCV_DIR}/lib/Debug/opencv_highgui310d.lib)
  EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E copy ${OpenCV_DIR}/lib/Debug/opencv_highgui310d.lib ${OpenCV_DIR}/lib/Debug/opencv_highgui300d.lib)
ENDIF()
IF(EXISTS ${OpenCV_DIR}/lib/Release/opencv_highgui310.lib)
  EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E copy ${OpenCV_DIR}/lib/Release/opencv_highgui310.lib ${OpenCV_DIR}/lib/Release/opencv_highgui300.lib)
ENDIF()

# End hack section
# --------------------------------------------------------------------------

# --------------------------------------------------------------------------
# OvrvisionPro library project
SET( OvrvisionPro_INCLUDE_DIRS #OpenCV directories are included via CMake target system 
  ${CMAKE_BINARY_DIR}
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/include
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/OpenCL/amd/include
  )

# --------------------------------------------------------------------------
# ArUco related files
SET( ArUco_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/ar_omp.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/arucofidmarkers.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/board.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/boarddetector.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/cameraparameters.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/chromaticmask.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/cvdrawingutils.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/highlyreliablemarkers.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/marker.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/markerdetector.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/subpixelcorner.cpp
  )

SET( ArUco_HEADERS
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/ar_omp.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/aruco.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/arucofidmarkers.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/board.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/boarddetector.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/cameraparameters.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/chromaticmask.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/cvdrawingutils.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/exports.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/highlyreliablemarkers.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/marker.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/markerdetector.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco/subpixelcorner.h
  )

# --------------------------------------------------------------------------
# CSharp related files
SET( OvrvisionProCSharp_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_csharp.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_csharp_unity.cpp
  )

# --------------------------------------------------------------------------
# OpenCL related files
SET( OvrvisionProOpenCL_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/OpenCL.cpp
  )

SET( OvrvisionProOpenCL_HEADERS
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/OpenCL_kernel.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/OvrvisionProCL.h
  )

# --------------------------------------------------------------------------
# Core lib files
SET( OvrvisionPro_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ar.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ar_metaio.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_calibration.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ds.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_pro.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_setting.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_tracking.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_v4l.cpp
  )

IF(APPLE)
  LIST(APPEND OvrvisionPro_SOURCES ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_avf.m)
ENDIF()

SET( OvrvisionPro_HEADERS
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_v4l.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_avf.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_calibration.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ds.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_handimage.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_setting.h
  )

ADD_LIBRARY(OvrvisionPro 
  ${ArUco_SOURCES} 
  ${ArUco_HEADERS} 
  ${OvrvisionProCSharp_SOURCES} 
  ${OvrvisionProOpenCL_SOURCES} 
  ${OvrvisionProOpenCL_HEADERS}
  ${OvrvisionPro_SOURCES}
  ${OvrvisionPro_HEADERS}
  )

source_group( "ArUco" FILES ${ArUco_SOURCES} ${ArUco_HEADERS} )
source_group( "C#" FILES ${OvrvisionProCSharp_SOURCES} )
source_group( "OpenCL" FILES ${OvrvisionProOpenCL_SOURCES} ${OvrvisionProOpenCL_HEADERS} )

target_link_libraries(OvrvisionPro PUBLIC ${OpenCV_LIBRARIES} ${OpenCL_LIBRARIES} ${OPENGL_LIBRARIES})
target_include_directories(OvrvisionPro PUBLIC ${OvrvisionPro_INCLUDE_DIRS})
target_compile_definitions(OvrvisionPro PRIVATE _OVRVISION_EXPORTS) # Build the library with DLL exports defined
target_compile_definitions(OvrvisionPro PUBLIC OPENCV_VERSION_3)

export(TARGETS OvrvisionPro FILE ${CMAKE_BINARY_DIR}/OvrvisionProTargets.cmake)

# Create a OvrvisionProConfig.cmake file for use from the build tree
CONFIGURE_FILE(OvrvisionProConfig.cmake.in
  "${CMAKE_BINARY_DIR}/OvrvisionProConfig.cmake" @ONLY)