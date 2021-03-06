cmake_minimum_required(VERSION 2.8.3)
project(transmission_interface)

if(USE_ROSBUILD)

  include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

  rosbuild_init()

  set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
  set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

  # Transmission Paraser Library
  rosbuild_add_library(${PROJECT_NAME}_parser src/transmission_parser.cpp)

  rosbuild_add_library(transmission_interface_loader
    src/transmission_loader.cpp           include/transmission_interface/transmission_loader.h
    src/transmission_interface_loader.cpp include/transmission_interface/transmission_interface_loader.h)
  target_link_libraries(transmission_interface_loader  ${PROJECT_NAME}_parser)

  rosbuild_add_library(transmission_interface_loader_plugins
    src/simple_transmission_loader.cpp           include/transmission_interface/simple_transmission_loader.h
    src/differential_transmission_loader.cpp     include/transmission_interface/differential_transmission_loader.h
    src/four_bar_linkage_transmission_loader.cpp include/transmission_interface/four_bar_linkage_transmission_loader.h
    src/joint_state_interface_provider.cpp       include/transmission_interface/joint_state_interface_provider.h
    src/position_joint_interface_provider.cpp    include/transmission_interface/position_joint_interface_provider.h
    src/velocity_joint_interface_provider.cpp    include/transmission_interface/velocity_joint_interface_provider.h
    src/effort_joint_interface_provider.cpp      include/transmission_interface/effort_joint_interface_provider.h)
  target_link_libraries(transmission_interface_loader_plugins transmission_interface_loader)

  rosbuild_add_gtest(simple_transmission_test                  test/simple_transmission_test.cpp)
  rosbuild_add_gtest(differential_transmission_test            test/differential_transmission_test.cpp)
  rosbuild_add_gtest(four_bar_linkage_transmission_test        test/four_bar_linkage_transmission_test.cpp)
  rosbuild_add_gtest(transmission_interface_test               test/transmission_interface_test.cpp)
  rosbuild_add_gtest(transmission_parser_test                  test/transmission_parser_test.cpp)
  rosbuild_add_gtest(simple_transmission_loader_test           test/simple_transmission_loader_test.cpp)
  rosbuild_add_gtest(differential_transmission_loader_test     test/differential_transmission_loader_test.cpp)
  rosbuild_add_gtest(four_bar_linkage_transmission_loader_test test/four_bar_linkage_transmission_loader_test.cpp)
  rosbuild_add_gtest(transmission_interface_loader_test        test/transmission_interface_loader_test.cpp)

  target_link_libraries(transmission_parser_test                  ${PROJECT_NAME}_parser)
  target_link_libraries(simple_transmission_loader_test           ${PROJECT_NAME}_parser)
  target_link_libraries(differential_transmission_loader_test     ${PROJECT_NAME}_parser)
  target_link_libraries(four_bar_linkage_transmission_loader_test ${PROJECT_NAME}_parser)
  target_link_libraries(transmission_interface_loader_test        transmission_interface_loader)

  # TODO: why is it explicitly needed???, without it linker fails.
  target_link_libraries(simple_transmission_test           pthread)
  target_link_libraries(differential_transmission_test     pthread)
  target_link_libraries(four_bar_linkage_transmission_test pthread)
  target_link_libraries(transmission_interface_test        pthread)

else()

  # Load catkin and all dependencies required for this package
  find_package(catkin REQUIRED
    hardware_interface
    cmake_modules
    pluginlib
    resource_retriever
    roscpp
  )

  # Include a custom cmake file for TinyXML
  find_package(TinyXML REQUIRED)

  # Declare a catkin package
  catkin_package(
    LIBRARIES
      ${PROJECT_NAME}_parser
    INCLUDE_DIRS
      include
    DEPENDS
      TinyXML
      pluginlib
      resource_retriever
      roscpp
  )

###########
## Build ##
###########

  # Build
  include_directories(include ${catkin_INCLUDE_DIRS} ${TinyXML_INCLUDE_DIRS})

  # Transmission parser Library
  add_library(${PROJECT_NAME}_parser
    src/transmission_parser.cpp include/transmission_interface/transmission_parser.h
  )
  target_link_libraries(${PROJECT_NAME}_parser ${catkin_LIBRARIES} ${TinyXML_LIBRARIES})

  # Transmission loader library
  add_library(transmission_interface_loader
    src/transmission_loader.cpp           include/transmission_interface/transmission_loader.h
    src/transmission_interface_loader.cpp include/transmission_interface/transmission_interface_loader.h
  )
  target_link_libraries(transmission_interface_loader
    ${PROJECT_NAME}_parser
    ${catkin_LIBRARIES}
    ${TinyXML_LIBRARIES})

  add_library(transmission_interface_loader_plugins
    src/simple_transmission_loader.cpp           include/transmission_interface/simple_transmission_loader.h
    src/differential_transmission_loader.cpp     include/transmission_interface/differential_transmission_loader.h
    src/four_bar_linkage_transmission_loader.cpp include/transmission_interface/four_bar_linkage_transmission_loader.h
    src/joint_state_interface_provider.cpp       include/transmission_interface/joint_state_interface_provider.h
    src/position_joint_interface_provider.cpp    include/transmission_interface/position_joint_interface_provider.h
    src/velocity_joint_interface_provider.cpp    include/transmission_interface/velocity_joint_interface_provider.h
    src/effort_joint_interface_provider.cpp      include/transmission_interface/effort_joint_interface_provider.h)
  target_link_libraries(transmission_interface_loader_plugins transmission_interface_loader)


#############
## Install ##
#############

  # Install
  install(DIRECTORY include/${PROJECT_NAME}/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

  # Mark executables and/or libraries for installation
  install(TARGETS ${PROJECT_NAME}_parser
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )

  # Mark cpp header files for installation
  install(DIRECTORY include/${PROJECT_NAME}/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
    FILES_MATCHING PATTERN "*.h"
    PATTERN ".svn" EXCLUDE
  )

#############
## Testing ##
#############

  catkin_add_gtest(simple_transmission_test           test/simple_transmission_test.cpp)
  catkin_add_gtest(differential_transmission_test     test/differential_transmission_test.cpp)
  catkin_add_gtest(four_bar_linkage_transmission_test test/four_bar_linkage_transmission_test.cpp)
  catkin_add_gtest(transmission_interface_test        test/transmission_interface_test.cpp)
  target_link_libraries(transmission_interface_test ${catkin_LIBRARIES} ${TinyXML_LIBRARIES})

  catkin_add_gtest(transmission_parser_test test/transmission_parser_test.cpp)
  target_link_libraries(transmission_parser_test ${PROJECT_NAME}_parser)

  catkin_add_gtest(simple_transmission_loader_test test/simple_transmission_loader_test.cpp)
  target_link_libraries(simple_transmission_loader_test ${PROJECT_NAME}_parser)

  catkin_add_gtest(differential_transmission_loader_test test/differential_transmission_loader_test.cpp)
  target_link_libraries(differential_transmission_loader_test ${PROJECT_NAME}_parser)

  catkin_add_gtest(four_bar_linkage_transmission_loader_test test/four_bar_linkage_transmission_loader_test.cpp)
  target_link_libraries(four_bar_linkage_transmission_loader_test ${PROJECT_NAME}_parser)

  catkin_add_gtest(transmission_interface_loader_test test/transmission_interface_loader_test.cpp)
  target_link_libraries(transmission_interface_loader_test ${PROJECT_NAME}_parser
                                                           transmission_interface_loader)

endif()
