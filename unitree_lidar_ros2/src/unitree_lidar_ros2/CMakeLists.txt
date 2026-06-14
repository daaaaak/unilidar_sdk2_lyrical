cmake_minimum_required(VERSION 3.16)
project(unitree_lidar_ros2)

if(POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(PCL REQUIRED COMPONENTS common io)
find_package(tf2_ros REQUIRED)
find_package(sensor_msgs REQUIRED)

set(AMENT_PREFIX_INCLUDE_DIRS)
if(DEFINED ENV{AMENT_PREFIX_PATH})
  string(REPLACE ":" ";" AMENT_PREFIX_PATH_LIST "$ENV{AMENT_PREFIX_PATH}")
  foreach(ament_prefix IN LISTS AMENT_PREFIX_PATH_LIST)
    if(EXISTS "${ament_prefix}/include")
      list(APPEND AMENT_PREFIX_INCLUDE_DIRS "${ament_prefix}/include")
      file(GLOB ament_package_include_dirs LIST_DIRECTORIES true "${ament_prefix}/include/*")
      foreach(ament_package_include_dir IN LISTS ament_package_include_dirs)
        if(IS_DIRECTORY "${ament_package_include_dir}")
          list(APPEND AMENT_PREFIX_INCLUDE_DIRS "${ament_package_include_dir}")
        endif()
      endforeach()
    endif()
  endforeach()
endif()

function(target_ros2_dependencies target)
  foreach(dependency IN LISTS ARGN)
    if(${dependency}_INCLUDE_DIRS)
      target_include_directories(${target} PUBLIC ${${dependency}_INCLUDE_DIRS})
    endif()

    if(${dependency}_DEFINITIONS)
      target_compile_definitions(${target} PUBLIC ${${dependency}_DEFINITIONS})
    endif()

    if(${dependency}_LIBRARIES)
      target_link_libraries(${target} PUBLIC ${${dependency}_LIBRARIES})
    endif()
  endforeach()
endfunction()

set(UNITREE_LIDAR_SDK_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../../../unitree_lidar_sdk")
set(UNITREE_LIDAR_SDK_LIB "${UNITREE_LIDAR_SDK_ROOT}/lib/${CMAKE_SYSTEM_PROCESSOR}/libunilidar_sdk2.a")

if(NOT EXISTS "${UNITREE_LIDAR_SDK_LIB}")
  message(FATAL_ERROR
    "Unitree LiDAR SDK library not found for ${CMAKE_SYSTEM_PROCESSOR}: "
    "${UNITREE_LIDAR_SDK_LIB}")
endif()

add_definitions(${PCL_DEFINITIONS})

add_executable(unitree_lidar_ros2_node src/unitree_lidar_ros2_node.cpp)

target_include_directories(unitree_lidar_ros2_node PRIVATE
  include
  ${UNITREE_LIDAR_SDK_ROOT}/include
  ${AMENT_PREFIX_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
)

target_compile_definitions(unitree_lidar_ros2_node PRIVATE ${PCL_DEFINITIONS})
target_link_directories(unitree_lidar_ros2_node PRIVATE ${PCL_LIBRARY_DIRS})

target_link_libraries(unitree_lidar_ros2_node PRIVATE
  ${PCL_LIBRARIES}
  ${UNITREE_LIDAR_SDK_LIB}
)

target_ros2_dependencies(
  unitree_lidar_ros2_node
  rclcpp
  std_msgs
  sensor_msgs
  geometry_msgs
  tf2_ros
  pcl_conversions
)

install(TARGETS
unitree_lidar_ros2_node
  DESTINATION lib/${PROJECT_NAME}
)

# install rviz file and launch file
install(FILES rviz/view.rviz launch/launch.py
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
