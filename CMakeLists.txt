SET(PROJECT_NAME VisionTools)

PROJECT(${PROJECT_NAME})
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/CMakeModules/")

#SET (OpenCV_DIR ${CMAKE_PREFIX_PATH}/share/OpenCV)

SET (CMAKE_VERBOSE_MAKEFILE ON)

IF( NOT CMAKE_BUILD_TYPE )
   SET( CMAKE_BUILD_TYPE Release )
ENDIF()

IF (CMAKE_COMPILER_IS_GNUCXX )
   SET(CMAKE_CXX_FLAGS_DEBUG  "-O0 -g")
   SET(CMAKE_CXX_FLAGS_RELEASE "-O3 -g ")

  ADD_DEFINITIONS("-Wall -Werror -Wno-unused-variable
                   -Wno-unused-but-set-variable -Wno-unknown-pragmas ")
ENDIF()


SET (INCLUDE_DIRS )

FIND_PACKAGE(OpenCV REQUIRED)
LIST(APPEND INCLUDE_DIRS ${OpenCV_INCLUDE_DIRS})
LIST(APPEND LIBS ${OpenCV_LIBS})
FIND_PACKAGE(Eigen3 REQUIRED)
LIST(APPEND INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})

LIST(APPEND INCLUDE_DIRS ${LIB_Sophus})
message(STATUS "sop ${LIB_Sophus}")

find_package(GLEW REQUIRED)

LIST(APPEND INCLUDE_DIRS ${GLEW_INCLUDE_DIR})
LIST(APPEND LIBS ${GLEW_LIBRARY})

SET (LIB_NAMES GL pangolin glut Sophus)

find_package (OpenGL REQUIRED)
if(OPENGL_FOUND)
 set(LIB_GL ${OPENGL_LIBRARIES})
endif(OPENGL_FOUND)

LIST(APPEND INCLUDE_DIRS ${FREEGLUT_INCLUDE_DIR})

FOREACH(lib ${LIB_NAMES})
  FIND_LIBRARY(LIB_${lib} ${lib})
  IF (LIB_${lib})
    MESSAGE(STATUS "found library '${lib}': ${LIB_${lib}}")
  ELSE (LIB_${lib})
    MESSAGE(FATAL_ERROR "cannot find library '${lib}' - this will not work ...")
  ENDIF (LIB_${lib})
  LIST(APPEND LIBS ${LIB_${lib}})
ENDFOREACH(lib)

include("${LIB_pangolin}/lib/cmake/Pangolin/PangolinConfig.cmake")
LIST(APPEND INCLUDE_DIRS ${Pangolin_INCLUDE_DIRS})
LIST(APPEND LIBS ${LIB_pangolin}/lib/pangolin.lib)

message(STATUS " pang lib ${Pangolin_LIBRARY}")
message(STATUS " pang lib2 ${Pangolin_LIBRARIES}")
message(STATUS " pang inc ${Pangolin_INCLUDE_DIRS}")

# this should be set by pangolin cmake file...
find_package(Boost COMPONENTS thread filesystem system REQUIRED QUIET)
message(STATUS " boost lib dir ${Boost_LIBRARY_DIR}")
link_directories(${Boost_LIBRARY_DIR})

SET (CLASSES  draw2d
              draw3d
              sample
              performance_monitor
              linear_camera)

SET (SOURCE_DIR "visiontools")

SET (SOURCES ${SOURCE_DIR}/gl_data.h
             ${SOURCE_DIR}/abstract_camera.h
             ${SOURCE_DIR}/accessor_macros.h
             ${SOURCE_DIR}/ringbuffer.h
             ${SOURCE_DIR}/stopwatch.h)

FOREACH(class ${CLASSES})
  LIST(APPEND SOURCES ${SOURCE_DIR}/${class}.cpp ${SOURCE_DIR}/${class}.h)
ENDFOREACH(class)

LINK_LIBRARIES (${PROJECT_NAME} ${LIBS})

INCLUDE_DIRECTORIES(${INCLUDE_DIRS})
ADD_LIBRARY(${PROJECT_NAME} STATIC ${SOURCES})


INSTALL(DIRECTORY visiontools DESTINATION ${CMAKE_INSTALL_PREFIX}/include FILES_MATCHING PATTERN "*.h" )
INSTALL(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib )