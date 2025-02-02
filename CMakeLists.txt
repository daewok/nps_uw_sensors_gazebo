cmake_minimum_required(VERSION 3.0.2)
project(nps_uw_sensors_gazebo)

## Make sure you have installed CUDA and include in paths
## LD_LIBRARY_PATH=LD_LIBRARY_PATH:/usr/local/cuda-11.1/lib64
## PATH=$PATH:/usr/local/cuda-11.1/bin
set(CMAKE_CUDA_COMPILER "nvcc")
enable_language(CUDA CXX)

if(NOT "${CMAKE_VERSION}" VERSION_LESS "3.16")
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
else()
    add_compile_options(-std=c++11)
endif()

find_package(catkin REQUIRED COMPONENTS
 tf
 gazebo_plugins
 acoustic_msgs)

find_package(gazebo REQUIRED)
find_package(roscpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(OpenCV REQUIRED)

find_package(CUDA REQUIRED)
include_directories(${CUDA_INCLUDE_DIRS})
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -arch=sm_60")

include_directories(${roscpp_INCLUDE_DIRS})
include_directories(${std_msgs_INCLUDE_DIRS})
include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
)
link_directories(${GAZEBO_LIBRARY_DIRS})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}")
set(SENSOR_ROS_PLUGINS_LIST "")

catkin_package(
  INCLUDE_DIRS
  LIBRARIES
  CATKIN_DEPENDS
  acoustic_msgs
 )

## Plugins

add_library(nps_image_sonar_ros_plugin
            src/gazebo_ros_image_sonar.cpp
            src/sonar_calculation_cuda.cu)
set_target_properties(nps_image_sonar_ros_plugin
                      PROPERTIES CUDA_SEPARABLE_COMPILATION ON)
target_link_libraries(nps_image_sonar_ros_plugin
                      ${OGRE_LIBRARIES} ${catkin_LIBRARIES}
                      ${CUDA_LIBRARIES}
                      ${CUDA_CUFFT_LIBRARIES})
add_dependencies(nps_image_sonar_ros_plugin ${catkin_EXPORTED_TARGETS})
list(APPEND SENSOR_ROS_PLUGINS_LIST nps_image_sonar_ros_plugin)

#add_library(nps_gazebo_ros_image_sonar_plugin
#            src/gazebo_ros_image_sonar.cpp
#            include/nps_uw_sensors_gazebo/gazebo_ros_image_sonar.hh)
#target_link_libraries(nps_gazebo_ros_image_sonar_plugin
#                      DepthCameraPlugin ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})

#add_library(nps_gazebo_ros_gpu_sonar_single_beam_plugin
#            src/nps_gazebo_ros_gpu_sonar_single_beam.cpp)
#target_link_libraries(nps_gazebo_ros_gpu_sonar_single_beam_plugin
#                      GpuRayPlugin ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})

#add_library(nps_gazebo_ros_depth_camera_sonar_single_beam_plugin
#            src/nps_gazebo_ros_depth_camera_sonar_single_beam.cpp
#            include/nps_uw_sensors_gazebo/nps_gazebo_ros_depth_camera_sonar_single_beam.hh)
#target_link_libraries(nps_gazebo_ros_depth_camera_sonar_single_beam_plugin
#                      DepthCameraPlugin ${catkin_LIBRARIES})

# Install plugins
install(
  TARGETS ${SENSOR_ROS_PLUGINS_LIST}
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

# for launch
install(DIRECTORY launch
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
        PATTERN "*~" EXCLUDE)

# for Python scripts
catkin_install_python(PROGRAMS
  scripts/depth_camera_sonar.py
  src/simple_box_motion.py
  src/simple_motion.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
