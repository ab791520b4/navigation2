cmake_minimum_required(VERSION 3.5)
project(nav2_smac_planner)

set(CMAKE_BUILD_TYPE Release) #Debug, Release

find_package(ament_cmake REQUIRED)
find_package(nav2_common REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)
find_package(std_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(nav2_util REQUIRED)
find_package(nav2_core REQUIRED)
find_package(nav2_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(nav2_costmap_2d REQUIRED)
find_package(pluginlib REQUIRED)
find_package(eigen3_cmake_module REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(ompl REQUIRED)
find_package(OpenMP REQUIRED)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")

add_compile_options(-O3 -Wextra -Wdeprecated -fPIC)

include_directories(
  include
  ${OMPL_INCLUDE_DIRS}
  ${OpenMP_INCLUDE_DIRS}
)

find_package(OpenMP)
if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

set(library_name nav2_smac_planner)

set(dependencies
  rclcpp
  rclcpp_action
  rclcpp_lifecycle
  std_msgs
  visualization_msgs
  nav2_util
  nav2_msgs
  nav_msgs
  geometry_msgs
  builtin_interfaces
  tf2_ros
  nav2_costmap_2d
  nav2_core
  pluginlib
  eigen3_cmake_module
)

# Hybrid plugin
add_library(${library_name} SHARED
  src/smac_planner_hybrid.cpp
  src/a_star.cpp
  src/node_hybrid.cpp
  src/costmap_downsampler.cpp
  src/node_2d.cpp
)

target_link_libraries(${library_name} ${OMPL_LIBRARIES} ${OpenMP_LIBRARIES}  OpenMP::OpenMP_CXX)
target_include_directories(${library_name} PUBLIC ${Eigen3_INCLUDE_DIRS})

ament_target_dependencies(${library_name}
  ${dependencies}
)

# 2D plugin
add_library(${library_name}_2d SHARED
  src/smac_planner_2d.cpp
  src/a_star.cpp
  src/node_hybrid.cpp
  src/costmap_downsampler.cpp
  src/node_2d.cpp
)

target_link_libraries(${library_name}_2d ${OMPL_LIBRARIES})
target_include_directories(${library_name}_2d PUBLIC ${Eigen3_INCLUDE_DIRS})

ament_target_dependencies(${library_name}_2d
  ${dependencies}
)

pluginlib_export_plugin_description_file(nav2_core smac_plugin.xml)
pluginlib_export_plugin_description_file(nav2_core smac_plugin_2d.xml)

install(TARGETS ${library_name} ${library_name}_2d
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY include/
  DESTINATION include/
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  find_package(ament_cmake_gtest REQUIRED)
  add_subdirectory(test)
endif()

ament_export_include_directories(include)
ament_export_libraries(${library_name} ${library_name}_2d)
ament_export_dependencies(${dependencies})
ament_package()
