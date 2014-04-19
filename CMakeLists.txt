project (osl CXX)

# Tweak build mode
if(NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "" FORCE)
endif()

if(NOT CMAKE_BUILD_TYPE MATCHES "^[Dd][Ee][Bb][Uu][Gg]")
        add_definitions(-DQT_NO_DEBUG_OUTPUT)
        set(CMAKE_BUILD_RELEASE TRUE)
else()
        set(CMAKE_BUILD_RELEASE FALSE)
endif()

# Installation directory
if(APPLE AND CMAKE_INSTALL_PREFIX MATCHES "/usr/local")
        set(CMAKE_INSTALL_PREFIX "/Applications")
endif()

message(STATUS "Building ${PROJECT_NAME} in ${CMAKE_BUILD_TYPE} mode")
cmake_minimum_required(VERSION 2.8.8)
set (CMAKE_VERBOSE_MAKEFILE on)

ENABLE_TESTING()

# Basic build options
if(CMAKE_COMPILER_IS_GNUCXX OR MINGW OR APPLE)
  #if(CMAKE_BUILD_RELEASE)
  #        add_definitions(-Werror)
  #endif()
  set (CMAKE_CXX_FLAGS "-Wall -D_REENTRANT ")
  set (CMAKE_CXX_FLAGS_DEBUG "-O0 -g -DDEBUG")
  set (CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG -funroll-loops -funswitch-loops -fpeel-loops --param large-function-growth=48000 --param inline-unit-growth=8000 --param max-inline-insns-single=8000 -fomit-frame-pointer")

  add_definitions(-DOSL_SMP=1 -DOSL_DFPN_SMP_SEARCH -DMORE_CHECKMATE_IF_CAPTURE_MAJOR -DUSE_TCMALLOC=0)
  add_definitions(-DBOOST_FILESYSTEM_VERSION=3) 
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR MINGW)
  add_definitions(-DUSE_BOOST_POOL_ALLOCATOR=1)
endif()

# Compiler specific build options
if(MINGW)
  set (CMAKE_CXX_FLAGS "-Wall -D_REENTRANT -D_MT -mthreads -mwin32 -msse4.1 -msse4.2 -mavx -static-libgcc -static-libstdc++ -DWIN32_LEAN_AND_MEAN -mstackrealign -fno-strict-aliasing -std=c++0x")
  add_definitions(-march=corei7-avx)
  add_definitions(-D_WIN32_WINNT=0x0500)

  set (BOOST_ROOT "$ENV{HOME}/gpsfish/local/src/boost_1_55_0")
  set (Boost_NO_SYSTEM_PATHS ON)
  find_library(Winsock2_LIBRARY ws2_32)
elseif (APPLE)
  set (CMAKE_CXX_COMPILER "/usr/local/bin/g++-4.8")
  #set (CMAKE_CXX_FLAGS "-Wall -D_REENTRANT -march=core2 -fno-common") # for shared library
  set (CMAKE_CXX_FLAGS "-Wall -D_REENTRANT -march=core2")
  set (CMAKE_OSX_ARCHITECTURES x86_64)
  set (CMAKE_OSX_SYSROOT "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.8.sdk")
  set (CMAKE_OSX_DEPLOYMENT_TARGET 10.6)

  set (BOOST_ROOT "/usr/local/Cellar/boost/1.54.0")
endif()

if (APPLE)
  add_definitions(-DOSL_HOME="/Applications/gpsfish.app/Contents/Resources")
else()
  add_definitions(-DOSL_HOME="${CMAKE_CURRENT_SOURCE_DIR}")
endif()

include_directories ("${PROJECT_SOURCE_DIR}/core")
#include_directories ("${PROJECT_SOURCE_DIR}/std")

## Pthreads
#find_package (Threads)
#if (Threads_FOUND)
#  message (STATUS "Threads: ${CMAKE_THREAD_LIBS_INIT}")
#else()
#  message (FATAL_ERROR "Threads not found")
#endif()

## Boost thing
#set (BOOST_ROOT "$ENV{HOME}/local/boost_1_36_0_binaries/")
set (Boost_USE_STATIC_LIBS ON)
set (Boost_USE_MULTITHREADED ON)
if (MINGW)
  find_package (Boost 1.55.0)
else()
  find_package (Boost 1.54.0)
endif()
if (Boost_FOUND)
  include_directories (${Boost_INCLUDE_DIRS})
  message (STATUS ${Boost_INCLUDE_DIRS})
else()
  message (FATAL_ERROR "Boost installation not found")
endif()

## Program SampleCounter
file(GLOB_RECURSE OSL_HEADERS  core/*.h core/*.tcc)
file(GLOB_RECURSE OSL_SRCS     core/osl/*.cc)
#file(GLOB_RECURSE OSL_HEADERS  std/*.h std/*.tcc)
#file(GLOB_RECURSE OSL_SRCS     std/osl/*.cc)
#file(GLOB_RECURSE EXCLUDE_SRCS lib/third_party/*.cc lib/ntesuki/*.cc)
#list(REMOVE_ITEM OSL_SRCS ${EXCLUDE_SRCS})
#message (STATUS "Headers: ${OSL_HEADERS}")
#message (STATUS "Sources: ${OSL_SRCS}")
#set (SampleAleriUDF_ALL_SRCS ${SampleAleriUDF_HEADERS} ${SampleAleriUDF_SRCS})
add_library (osl-static STATIC ${OSL_SRCS})
SET_TARGET_PROPERTIES(osl-static PROPERTIES OUTPUT_NAME "osl")
SET_TARGET_PROPERTIES(osl-static PROPERTIES PREFIX "lib")
if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  add_library (osl SHARED ${OSL_SRCS})
  target_link_libraries (osl ${CMAKE_THREAD_LIBS_INIT})
endif()

