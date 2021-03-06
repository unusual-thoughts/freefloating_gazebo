cmake_minimum_required(VERSION 2.8.3)
project(freefloating_gazebo)
set(CMAKE_CXX_FLAGS "-std=c++0x ${CMAKE_CXX_FLAGS}")
find_package(catkin REQUIRED COMPONENTS
  gazebo_ros
  urdf
  control_toolbox
  rostime
  message_generation
)

find_package(gazebo REQUIRED)
find_package(Eigen3)
if(NOT EIGEN3_FOUND)
  # Fallback to cmake_modules
  find_package(cmake_modules REQUIRED)
  find_package(Eigen REQUIRED)
  set(EIGEN3_INCLUDE_DIRS ${EIGEN_INCLUDE_DIRS})
else()
  set(EIGEN3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})
endif()

add_message_files(
    FILES
    BodySetpoint.msg
)

# build messages
generate_messages(
    DEPENDENCIES geometry_msgs
    )

catkin_package(
 INCLUDE_DIRS include
 LIBRARIES ${PROJECT_NAME} 
 DEPENDS
    gazebo_ros
    control_toolbox
    rostime
)

link_directories(${GAZEBO_LIBRARY_DIRS})
include_directories(include ${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS} ${GAZEBO_INCLUDE_DIRS} ${EIGEN3_INCLUDE_DIRS})

# world plugin to simulate buoyancy and viscous force
add_library(freefloating_gazebo_fluid src/freefloating_gazebo_fluid.cpp include/freefloating_gazebo/freefloating_gazebo_fluid.h)
target_link_libraries(freefloating_gazebo_fluid ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})
add_dependencies(freefloating_gazebo_fluid freefloating_gazebo_generate_messages_cpp)

# model plugin subscribes to body and joint efforts and applies them in Gazebo
add_library(freefloating_gazebo_control src/freefloating_gazebo_control.cpp include/freefloating_gazebo/freefloating_gazebo_control.h)
target_link_libraries(freefloating_gazebo_control ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})
add_dependencies(freefloating_gazebo_control freefloating_gazebo_generate_messages_cpp)

# node to perform PID control for body (thrusters) and joints
add_executable(pid_control src/freefloating_pids_main.cpp
src/freefloating_pids.cpp include/freefloating_gazebo/freefloating_pids.h
src/freefloating_pids_body.cpp include/freefloating_gazebo/freefloating_pids_body.h
src/freefloating_pids_joint.cpp include/freefloating_gazebo/freefloating_pids_joint.h)
target_link_libraries(pid_control ${GAZEBO_LIBRARIES} ${catkin_LIBRARIES})
add_dependencies(pid_control freefloating_gazebo_generate_messages_cpp)


