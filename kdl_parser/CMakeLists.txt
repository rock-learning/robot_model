cmake_minimum_required(VERSION 2.8.3)

project(kdl_parser)

set(CMAKE_INSTALL_LIBDIR lib)

find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIR})
link_directories(${Boost_LIBRARY_DIRS})

find_package(urdfdom_headers_approxik REQUIRED)
include_directories(include ${urdfdom_headers_approxik_INCLUDE_DIRS})

find_package(urdfdom REQUIRED)
include_directories(include ${urdfdom_INCLUDE_DIRS})
link_directories(${urdfdom_LIBRARY_DIRS})

find_package(console_bridge REQUIRED)
include_directories(include ${console_bridge_INCLUDE_DIRS})
link_directories(${console_bridge_LIBRARY_DIRS})

find_package(orocos_kdl REQUIRED)
include_directories(include ${orocos_kdl_INCLUDE_DIRS})
link_directories(${orocos_kdl_LIBRARY_DIRS})

find_library(KDL_LIBRARY REQUIRED NAMES orocos-kdl HINTS ${orocos_kdl_LIBRARY_DIRS})

add_library(${PROJECT_NAME} SHARED src/kdl_parser.cpp)
target_link_libraries(${PROJECT_NAME}
  ${urdfdom_LIBRARIES} ${console_bridge_LIBRARIES} ${orocos_kdl_LIBRARIES}
)

add_executable(check_kdl_parser src/check_kdl_parser.cpp)
target_link_libraries(check_kdl_parser ${PROJECT_NAME})

install(TARGETS ${PROJECT_NAME} DESTINATION lib)
install(DIRECTORY include/${PROJECT_NAME}/ DESTINATION include/${PROJECT_NAME})

if(WIN32 AND NOT CYGWIN)
  set(CMAKE_CONFIG_INSTALL_DIR CMake)
else()
  set(CMAKE_CONFIG_INSTALL_DIR share/${PROJECT_NAME}/cmake/)
endif()

set(PKG_NAME ${PROJECT_NAME})
set(PKG_LIBRARIES kdl_parser)
set(PKG_DEPENDS urdfdom_headers_approxik urdfdom console_bridge orocos_kdl)
set(cmake_conf_file "cmake/kdl_parser-config.cmake")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${cmake_conf_file}.in" "${CMAKE_BINARY_DIR}/${cmake_conf_file}" @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/${cmake_conf_file}
  DESTINATION ${CMAKE_CONFIG_INSTALL_DIR} COMPONENT cmake)

# Make the package config file
if (NOT MSVC)
  set(pkg_conf_file "kdl_parser.pc")
  configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/pkgconfig/${pkg_conf_file}.in" "${CMAKE_BINARY_DIR}/${pkg_conf_file}" @ONLY)
  install(FILES "${CMAKE_BINARY_DIR}/${pkg_conf_file}" DESTINATION lib/pkgconfig/ COMPONENT pkgconfig)
endif()

message(STATUS "Configuration successful. Type make install to install kdl_parser.")
