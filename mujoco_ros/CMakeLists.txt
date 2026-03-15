cmake_minimum_required(VERSION 2.8.3)
project(mujoco_ros)

add_compile_options(-Ofast -std=c++11)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  mujoco_ros_msgs
  eigen_conversions
)

find_package(tocabi_lib COMPONENTS)
IF(tocabi_lib_FOUND)
 message(" -- shm mode enabled" )
   ADD_DEFINITIONS(-DCOMPILE_SHAREDMEMORY)
ENDIF()

set(MUJOCO_PATH_CANDIDATES
  ${CMAKE_CURRENT_SOURCE_DIR}/../mujoco-2.1.1
  ${CMAKE_CURRENT_SOURCE_DIR}/../mujoco211
)

foreach(candidate ${MUJOCO_PATH_CANDIDATES})
  if(EXISTS ${candidate}/include/mujoco.h)
    set(MUJOCO_PATH ${candidate})
    break()
  endif()
endforeach()

if(NOT MUJOCO_PATH)
  message(FATAL_ERROR "MuJoCo 2.1.1 was not found. Expected one of: ${MUJOCO_PATH_CANDIDATES}")
endif()

set(MUJOCO_LIBRARY_DIR ${MUJOCO_PATH}/lib)

message("MuJoCo path: " ${MUJOCO_PATH})

catkin_package(
DEPENDS   mujoco_ros_msgs
)


include_directories(
  include
  ${MUJOCO_PATH}/include
  ${MUJOCO_PATH}/sample
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
)

link_directories(
  ${MUJOCO_LIBRARY_DIR}
)

IF(tocabi_lib_FOUND)
include_directories(
  ${tocabi_lib_INCLUDE_DIRS}
)
ENDIF()

find_library(MUJOCO_LIB
  NAMES mujoco
  PATHS ${MUJOCO_LIBRARY_DIR}
  NO_DEFAULT_PATH
)

find_library(MUJOCO_GLEW_LIB
  NAMES glew
  PATHS ${MUJOCO_LIBRARY_DIR}
  NO_DEFAULT_PATH
)

find_path(GLFW_INCLUDE_DIR
  NAMES GLFW/glfw3.h
)

find_library(GLFW_LIB
  NAMES glfw glfw3
)

if(NOT MUJOCO_LIB)
  message(FATAL_ERROR "MuJoCo shared library was not found in ${MUJOCO_LIBRARY_DIR}")
endif()

if(NOT MUJOCO_GLEW_LIB)
  message(FATAL_ERROR "MuJoCo GLEW shared library was not found in ${MUJOCO_LIBRARY_DIR}")
endif()

if(NOT GLFW_LIB)
  message(FATAL_ERROR "GLFW library was not found. Install a system GLFW package such as libglfw3-dev.")
endif()

if(NOT GLFW_INCLUDE_DIR)
  message(FATAL_ERROR "GLFW headers were not found. Install a system GLFW development package such as libglfw3-dev.")
endif()

include_directories(
  ${GLFW_INCLUDE_DIR}
)

set(UI_SRC ${MUJOCO_PATH}/sample/uitools.c)

add_library(mujoco_ros_lib
    ${UI_SRC}
    src/mjros.cpp
    )

add_executable(mujoco_ros src/main.cpp)

target_link_libraries(mujoco_ros
   mujoco_ros_lib

   ${catkin_LIBRARIES}
   ${MUJOCO_LIB}
   GL
   ${MUJOCO_GLEW_LIB}
   ${GLFW_LIB}
   #GLU
   #glut
 )

set_target_properties(mujoco_ros PROPERTIES
  LINK_FLAGS "-Wl,-rpath,${MUJOCO_LIBRARY_DIR}"
)

add_dependencies(mujoco_ros_lib ${catkin_EXPORTED_TARGETS})
add_dependencies(mujoco_ros ${catkin_EXPORTED_TARGETS})
