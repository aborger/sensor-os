cmake_minimum_required(VERSION 2.8.3)
project(ros_sensor_driver)


set(GPIO_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../JetsonGPIO)
set(SCHEDULER_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../scheduler)
set(I2C_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../libi2c)

include_directories(${GPIO_PATH}/include)
add_library(JetsonGPIO STATIC 
  ${GPIO_PATH}/src/JetsonGPIO.cpp
  ${GPIO_PATH}/src/PythonFunctions.cpp
  ${GPIO_PATH}/src/gpio_pin_data.cpp
)

include_directories(${I2C_PATH}/include)
add_library(I2C STATIC
  ${I2C_PATH}/src/i2c.c
)



set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")

include_directories(${SCHEDULER_PATH}/include)
add_library(Scheduler STATIC
  ${SCHEDULER_PATH}/src/scheduler.cpp
  ${SCHEDULER_PATH}/src/process.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/programs.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/I2C_program.cpp
)





find_package(catkin REQUIRED 
  COMPONENTS 
  roscpp 
  rospy 
  std_msgs
  )

catkin_package()

include_directories(include
  ${catkin_INCLUDE_DIRS} 
)

catkin_install_python(PROGRAMS
   nodes/drive_pub
   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

add_executable(talker src/talker.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})

add_executable(listener src/listener.cpp)
target_link_libraries(listener ${catkin_LIBRARIES})

add_executable(sensor nodes/i2c_sensor.cpp)
target_link_libraries(sensor ${catkin_LIBRARIES} I2C)

add_executable(driver nodes/driver_node.cpp)
target_link_libraries(driver ${catkin_LIBRARIES} Scheduler JetsonGPIO I2C)
