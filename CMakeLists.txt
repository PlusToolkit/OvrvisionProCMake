project(OvrvisionPro)
cmake_minimum_required(VERSION 3.0)

OPTION(BUILD_SHARED_LIBS "Build shared libraries" ON)

# --------------------------------------------------------------------------
# Configure output paths for libraries and executables.
IF(NOT DEFINED CMAKE_RUNTIME_OUTPUT_DIRECTORY)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
ENDIF()

IF(NOT DEFINED CMAKE_LIBRARY_OUTPUT_DIRECTORY)
  SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
ENDIF()

IF(NOT DEFINED CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
  SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
ENDIF()

# --------------------------------------------------------------------------
# Ensure all dependencies are available
FIND_PACKAGE(OpenCV REQUIRED NO_MODULE COMPONENTS opencv_core opencv_imgproc opencv_calib3d)
IF(OpenCV_VERSION VERSION_LESS 3.1.0)
  MESSAGE(FATAL_ERROR "OpenCV >= 3.1.0 required for OvrvisionPro")
ELSE()
  MESSAGE(STATUS "Using OpenCV ${OpenCV_VERSION}")
ENDIF()

FIND_PACKAGE(OpenCL REQUIRED)
FIND_PACKAGE(OpenGL REQUIRED)

# --------------------------------------------------------------------------
# OpenCV dll section
FOREACH(target ${OpenCV_LIBRARIES})
  IF(NOT TARGET ${target})
    CONTINUE()
  ENDIF()

  GET_TARGET_PROPERTY(LIB_TYPE ${target} TYPE)

  IF(${LIB_TYPE} STREQUAL "SHARED_LIBRARY" AND ${CMAKE_GENERATOR} MATCHES "Visual Studio")
    GET_TARGET_PROPERTY(LIB_DEBUG_FILE ${target} IMPORTED_LOCATION_DEBUG)
    GET_TARGET_PROPERTY(LIB_RELEASE_FILE ${target} IMPORTED_LOCATION_RELEASE)

    IF(NOT LIB_DEBUG_FILE AND NOT LIB_RELEASE_FILE)
      GET_TARGET_PROPERTY(LIB_FILE ${target} IMPORTED_LOCATION)
      IF("${LIB_FILE}" STREQUAL "")
        MESSAGE(STATUS "Unable to copy target ${target} to runtime directory. No file associated with target.")
      ELSE()
        SET(LIB_RELEASE_FILE ${LIB_FILE})
        SET(LIB_DEBUG_FILE ${LIB_FILE})
      ENDIF()
    ENDIF()

    IF( EXISTS ${LIB_RELEASE_FILE} )
      FILE(COPY ${LIB_RELEASE_FILE} DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/Release)
    ENDIF()
    IF( EXISTS ${LIB_DEBUG_FILE} )
      FILE(COPY ${LIB_DEBUG_FILE} DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/Debug)
      # Also copy the .pdb if it exists
      STRING(REPLACE "${CMAKE_SHARED_LIBRARY_SUFFIX}" ".pdb" LIB_DEBUG_PDB_FILE ${LIB_DEBUG_FILE})
      IF( EXISTS ${LIB_DEBUG_PDB_FILE})
        FILE(COPY ${LIB_DEBUG_PDB_FILE} DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/Debug)
      ENDIF()
    ENDIF()
  ENDIF()
ENDFOREACH()

# --------------------------------------------------------------------------
# OvrvisionPro library project
SET(OvrvisionPro_INCLUDE_DIRS #OpenCV directories are included via CMake target system 
  ${CMAKE_BINARY_DIR}
  ${OpenCL_INCLUDE_DIRS}
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/include
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/3rdparty/aruco
  )

# --------------------------------------------------------------------------
# ArUco related files
SET(ArUco_SOURCES
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

SET(ArUco_HEADERS
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
SET(OvrvisionProCSharp_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_csharp.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_csharp_unity.cpp
  )

# --------------------------------------------------------------------------
# OpenCL related files
SET(OvrvisionProOpenCL_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/OpenCL.cpp
  )

SET(OvrvisionProOpenCL_HEADERS
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/OpenCL_kernel.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/OvrvisionProCL.h
  )

# --------------------------------------------------------------------------
# Core lib files
SET(OvrvisionPro_SOURCES
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ar.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ar_metaio.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_calibration.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_pro.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_setting.cpp
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_tracking.cpp
  )

IF(WIN32)
  LIST(APPEND OvrvisionPro_SOURCES ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ds.cpp)
ELSEIF(APPLE)
  LIST(APPEND OvrvisionPro_SOURCES ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_avf.m)
ELSEIF(UNIX)
  LIST(APPEND OvrvisionPro_SOURCES ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_v4l.cpp)
ENDIF()

SET(OvrvisionPro_HEADERS
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_calibration.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_handimage.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_setting.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/include/ovrvision_ar.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/include/ovrvision_pro.h
  ${CMAKE_SOURCE_DIR}/OvrvisionPro/include/ovrvision_tracking.h
  )

IF(WIN32)
  LIST(APPEND OvrvisionPro_HEADERS ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_ds.h)
ELSEIF(APPLE)
  LIST(APPEND OvrvisionPro_HEADERS ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_avf.h)
  LIST(APPEND OvrvisionPro_HEADERS ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_avf.m)
ELSEIF(UNIX)
  LIST(APPEND OvrvisionPro_HEADERS ${CMAKE_SOURCE_DIR}/OvrvisionPro/src/lib_src/ovrvision_v4l.h)
ENDIF()

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

SET(OvrvisionPro_LIBS
  ${OpenCV_LIBRARIES}
  ${OpenCL_LIBRARIES}
  ${OPENGL_LIBRARIES}
  )

IF(WIN32)
  LIST(APPEND OvrvisionPro_LIBS 
    opencv_highgui
    Strmiids.lib
    )
ELSEIF(APPLE)
  LIST(APPEND OvrvisionPro_LIBS )
ELSEIF(UNIX)
  LIST(APPEND OvrvisionPro_LIBS )
ENDIF()

target_link_libraries(OvrvisionPro PUBLIC ${OvrvisionPro_LIBS} )
target_include_directories(OvrvisionPro PUBLIC ${OvrvisionPro_INCLUDE_DIRS})
target_compile_definitions(OvrvisionPro PRIVATE _OVRVISION_EXPORTS) # Build the library with DLL exports defined
target_compile_definitions(OvrvisionPro PUBLIC OPENCV_VERSION_3)

export(TARGETS OvrvisionPro FILE ${CMAKE_BINARY_DIR}/OvrvisionProTargets.cmake)

INSTALL(TARGETS OvrvisionPro
  RUNTIME DESTINATION bin CONFIGURATIONS Release COMPONENT RuntimeLibraries
  LIBRARY DESTINATION lib CONFIGURATIONS Release COMPONENT RuntimeLibraries
  ARCHIVE DESTINATION lib CONFIGURATIONS Release COMPONENT Development
  )
INSTALL(FILES ${ArUco_HEADERS} ${OvrvisionProOpenCL_HEADERS} ${OvrvisionPro_HEADERS}
  DESTINATION include
  COMPONENT Includes
  )

# Create a OvrvisionProConfig.cmake file for use from the build tree
CONFIGURE_FILE(OvrvisionProConfig.cmake.in
  "${CMAKE_BINARY_DIR}/OvrvisionProConfig.cmake" @ONLY)