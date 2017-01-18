# remove *.skeleton.cpp before run cmake

cmake_minimum_required(VERSION 2.8)
project(thrift-cpp-demo)


include_directories("/usr/include/boost/")

#Make sure gen-cpp files can be included
include_directories("${CMAKE_CURRENT_BINARY_DIR}")
include_directories("${CMAKE_CURRENT_BINARY_DIR}/gen-cpp")
#include_directories("${PROJECT_SOURCE_DIR}/lib/cpp/src")

# SET(CMAKE_CXX_FLAGS_RELEASE  "-lpthread")

set(SERVER_SOURCE "CppServer.cpp")
set(CLIENT_SOURCE "CppClient.cpp")

file(GLOB thrift_gen_cpp_SOURCES "gen-cpp/*.cpp")
# list(REMOVE_ITEM thrift_gen_cpp_SOURCES "/root/dl-data/thrift-cpp/gen-cpp/SharedService_server.skeleton.cpp")
message(STATUS "haha ${thrift_gen_cpp_SOURCES}")

add_library(thrift_gen_cpp STATIC ${thrift_gen_cpp_SOURCES})
target_link_libraries(thrift_gen_cpp /usr/local/lib/libthrift.a)
target_link_libraries(thrift_gen_cpp pthread)


add_executable(${SERVER_SOURCE} CppServer.cpp)
target_link_libraries(${SERVER_SOURCE} thrift_gen_cpp)
target_link_libraries(${SERVER_SOURCE} /usr/local/lib/libthrift.a)
# LINK_AGAINST_THRIFT_LIBRARY(${SERVER_SOURCE} thrift)
target_link_libraries(${SERVER_SOURCE} ${ZLIB_LIBRARIES})
target_link_libraries(${SERVER_SOURCE} pthread)

add_executable(${CLIENT_SOURCE} CppClient.cpp)
target_link_libraries(${CLIENT_SOURCE} thrift_gen_cpp)
target_link_libraries(${CLIENT_SOURCE} /usr/local/lib/libthrift.a)
# LINK_AGAINST_THRIFT_LIBRARY(${CLIENT_SOURCE} thrift)
target_link_libraries(${CLIENT_SOURCE} ${ZLIB_LIBRARIES})
target_link_libraries(${CLIENT_SOURCE} pthread)
