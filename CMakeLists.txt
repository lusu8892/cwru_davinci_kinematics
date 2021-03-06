cmake_minimum_required(VERSION 2.8.3)
project(cwru_davinci_kinematics)

#This is needed as part of the migration to ros jade and later
find_package(Eigen3)

# this is depreciated but still necessary for now.
if(NOT EIGEN3_FOUND)
  # Fallback to cmake_modules
  find_package(cmake_modules REQUIRED)
  find_package(Eigen REQUIRED)
  set(Eigen3_INCLUDE_DIRS ${EIGEN_INCLUDE_DIR})
  set(Eigen3_DEP Eigen)
  # message("Found Eigen ${EIGEN_INCLUDE_DIR}")
  # Possibly map additional variables to the EIGEN3_ prefix.
else()
  set(Eigen3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})
  set(Eigen3_DEP Eigen3)
  # message("Found Eigen3 ${EIGEN3_INCLUDE_DIR}")
endif()

add_definitions(${EIGEN_DEFINITIONS})

find_package(catkin REQUIRED COMPONENTS roscpp roslint tf tf2 tf2_eigen)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=gnu++0x")

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${catkin_INCLUDES}
  ${Eigen3_INCLUDE_DIRS}
)

catkin_package(
	DEPENDS ${Eigen3_DEP}
	LIBRARIES davinci_kinematics
	CATKIN_DEPENDS roscpp
	INCLUDE_DIRS include ${EIGEN3_INCLUDE_DIRS}
)


# Libraries

add_library(davinci_kinematics src/davinci_fwd_kinematics.cpp src/davinci_inv_kinematics.cpp)

target_link_libraries(davinci_kinematics
  ${catkin_LIBRARIES}
  davinci_kinematic_definitions
  )

add_library(davinci_kinematic_definitions src/davinci_kinematic_definitions.cpp)


# Examples

add_executable(jacobian_demo examples/davinci_jacobian_demo.cpp)

target_link_libraries(jacobian_demo
  davinci_kinematics
  ${catkin_LIBRARIES}
)

add_executable(forward_demo examples/davinci_fwd_kinematics_demo.cpp)

target_link_libraries(forward_demo
  davinci_kinematics
  davinci_kinematic_definitions
  ${catkin_LIBRARIES}
)

add_executable(ik_demo examples/davinci_inv_kinematics_demo.cpp)

target_link_libraries(ik_demo
  davinci_kinematics
  davinci_kinematic_definitions
  ${catkin_LIBRARIES}
)


file(GLOB_RECURSE SRC_FILES_CPP src/*.cpp)
file(GLOB_RECURSE TEST_FILES_CPP test/*.cpp)
file(GLOB_RECURSE INCLUDE_FILES_H include/*.h)
file(GLOB_RECURSE EXAMPLE_FILES_CPP examples/*.cpp)
roslint_cpp(${SRC_FILES_CPP} ${TEST_FILES_CPP} ${INCLUDE_FILES_H} ${EXAMPLE_FILES_CPP})


# Test lists:
# TODO(rcj), convert these tests into unit testing for code verification.

roslint_add_test()

catkin_add_gtest(full_kinematics_test test/full_kinematics.cpp)
target_link_libraries(full_kinematics_test
  davinci_kinematics
  davinci_kinematic_definitions
  ${catkin_LIBRARIES}
)