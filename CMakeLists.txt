cmake_minimum_required(VERSION 3.8)
project(hatchbed_localization)

find_package(ament_cmake REQUIRED)
find_package(eigen3_cmake_module REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(hatchbed_common REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_msgs REQUIRED)
find_package(tf2_ros REQUIRED)

include_directories(
  include
)

add_library(${PROJECT_NAME} SHARED
  src/covariance_util.cpp
  src/imu_to_twist.cpp
  src/odom_tf_broadcast.cpp
  src/odom_to_twist.cpp
  src/reframe_odom.cpp
  src/tf_reroot.cpp
)
target_link_libraries(${PROJECT_NAME}
  Eigen3::Eigen
)

set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 17)

ament_target_dependencies(${PROJECT_NAME}
  geometry_msgs
  hatchbed_common
  nav_msgs
  rclcpp
  rclcpp_components
  sensor_msgs
  tf2
  tf2_msgs
  tf2_ros
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "hatchbed_localization::ImuToTwist"
  EXECUTABLE imu_to_twist
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "hatchbed_localization::OdomTfBroadcast"
  EXECUTABLE odom_tf_broadcast
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "hatchbed_localization::OdomToTwist"
  EXECUTABLE odom_to_twist
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "hatchbed_localization::ReframeOdom"
  EXECUTABLE reframe_odom
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "hatchbed_localization::TfReroot"
  EXECUTABLE tf_reroot
)

install(TARGETS
  ${PROJECT_NAME}
  imu_to_twist
  odom_tf_broadcast
  odom_to_twist
  reframe_odom
  tf_reroot
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY include/
  DESTINATION include/
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  list(APPEND AMENT_LINT_AUTO_EXCLUDE ament_cmake_cpplint ament_cmake_uncrustify)
  ament_lint_auto_find_test_dependencies()
  find_package(ament_cmake_cpplint REQUIRED)
  ament_cpplint(
    FILTERS "-build/include_order"
    MAX_LINE_LENGTH 100)

  find_package(ament_cmake_gtest REQUIRED)
  ament_add_gtest(test_covariance_util
      test/test_covariance_util.cpp)

  target_link_libraries(test_covariance_util
      ${PROJECT_NAME})
endif()

ament_package()
