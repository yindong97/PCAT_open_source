cmake_minimum_required(VERSION 2.8.11)
project(rviz_cloud_annotation)

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  interactive_markers
  pcl_ros
  pcl_conversions
  roscpp
  tf
  visualization_msgs
  geometry_msgs
  eigen_conversions
  cmake_modules
  std_msgs
  rviz
)
find_package(OpenCV)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED)
find_package(PCL 1.8)
message(WARNING "PCL 1.8 found")

set(MAYBE_PCL_COMMON_COLORS "")
if (NOT ${PCL_FOUND})
  message(WARNING "PCL 1.8 not found, attempting 1.7...")
  find_package(PCL 1.7 REQUIRED)
  include_directories(${CMAKE_CURRENT_SOURCE_DIR}/pcl_include)
  set(MAYBE_PCL_COMMON_COLORS "${CMAKE_CURRENT_SOURCE_DIR}/pcl_include/colors.cpp")
endif()

find_package(Qt5Widgets REQUIRED)
set(CMAKE_CXX_STANDARD 11) # C++11...
set(CMAKE_AUTOMOC ON)
add_definitions(-DQT_NO_KEYWORDS)

add_definitions(${PCL_DEFINITIONS})
link_directories(${PCL_LIBRARY_DIRS})

################################################
## Declare ROS messages, services and actions ##
################################################

add_message_files(
  FILES
  UndoRedoState.msg
  RectangleSelectionViewport.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
  geometry_msgs
)

###################################
## catkin specific configuration ##
###################################

catkin_package()

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${Eigen3_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
)

add_executable(rviz_cloud_annotation_node
  src/rviz_cloud_annotation.cpp
  ${MAYBE_PCL_COMMON_COLORS}
  )

add_dependencies(rviz_cloud_annotation_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

target_link_libraries(
  rviz_cloud_annotation_node
  rviz_cloud_annotation_com
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
  ${Eigen3_LIBRARIES}
  ${catkin_LIBRARIES}
  )


set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

add_library(rviz_cloud_annotation_plugin
  src/rviz_cloud_annotation_plugin.cpp
  src/rviz_select_tool.cpp
  ${MAYBE_PCL_COMMON_COLORS}
)

add_dependencies(rviz_cloud_annotation_plugin ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

target_link_libraries(rviz_cloud_annotation_plugin
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
  ${Eigen3_LIBRARIES}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBRARIES}
  Qt5::Widgets
)


add_library(rviz_cloud_annotation_com
  src/rviz_cloud_annotation_points.cpp
  src/rviz_cloud_annotation_points_io.cpp
  src/rviz_cloud_annotation_point_plane.cpp
  src/rviz_cloud_annotation_class.cpp
  src/rviz_cloud_annotation_undo.cpp
  src/point_neighborhood.cpp
  src/point_neighborhood_search.cpp
  src/point_cloud_plane_curves_extract.cpp
  src/point_cloud_plane_params.cpp
  ${MAYBE_PCL_COMMON_COLORS}
)


add_dependencies(rviz_cloud_annotation_com ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

target_link_libraries(rviz_cloud_annotation_com
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
  ${Eigen3_LIBRARIES}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBRARIES}
  Qt5::Widgets
)

