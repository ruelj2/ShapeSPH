cmake_minimum_required(VERSION 3.10)

#-------------------------------------------------------------------------------
# Project Definitions
#-------------------------------------------------------------------------------
project(ShapeSPH
        DESCRIPTION "Shape Spherical Harmonics"
        LANGUAGES CXX C)
set(PROJECT_VERSION_MAJOR "0")
set(PROJECT_VERSION_MINOR "1")
set(PROJECT_VERSION_PATCH "0")
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Wno-alloc-size-larger-than
-Wno-deprecated-declarations -Wno-unused-variable -Wno-stringop-truncation")

# Optimization Config
set(CMAKE_CXX_FLAGS "-Wall -Wextra")
set(CMAKE_CXX_FLAGS_DEBUG "-g")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

# Do not allow to build in main repo
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
if(EXISTS "${LOC_PATH}")
    message(FATAL_ERROR "You cannot build in a source directory (or any directory with a CMakeLists.txt file). Please make a build subdirectory. Feel free to remove CMakeCache.txt and CMakeFiles.")
endif()

# Set the default build type
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
            STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            "Debug" "Release")
endif()


#-------------------------------------------------------------------------------
# Setup OpenMP
#-------------------------------------------------------------------------------
find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

#-------------------------------------------------------------------------------
# External FFTW Library
#-------------------------------------------------------------------------------
configure_file(extern/downloadFindFFTW.cmake.in findFFTW-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findFFTW-download )
if(result)
    message(FATAL_ERROR "CMake step for findFFTW failed: ${result}")
else()
    message("CMake step for findFFTW completed (${result}).")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findFFTW-download )
if(result)
    message(FATAL_ERROR "Build step for findFFTW failed: ${result}")
endif()

set(findFFTW_DIR ${CMAKE_CURRENT_BINARY_DIR}/findFFTW-src)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${findFFTW_DIR}")
find_package(FFTW)

#-------------------------------------------------------------------------------
# Configuration
#-------------------------------------------------------------------------------
add_definitions(-DINLINE_CMD_LINE_PARSER)
MESSAGE(STATUS message:"${FFTW_INCLUDE_DIRS}")

#-------------------------------------------------------------------------------
# Build SOFT
#-------------------------------------------------------------------------------
file(GLOB soft_SRC Soft10/*.cpp)
add_library(soft SHARED ${soft_SRC})
target_Include_directories(soft PUBLIC Soft10/)

#-------------------------------------------------------------------------------
# Build PLY
#-------------------------------------------------------------------------------
add_library(ply SHARED Include/Util/PlyFile.cpp)
target_Include_directories(ply PUBLIC Include/)

#-------------------------------------------------------------------------------
# Build Util (.cpp)
#-------------------------------------------------------------------------------
add_library(util SHARED Include/Util/Util.cpp)
target_Include_directories(util PUBLIC Include/)

#-------------------------------------------------------------------------------
# Build ShapeAlign
#-------------------------------------------------------------------------------
add_executable(ShapeAlign ShapeAlign/ShapeAlign.cpp)
target_link_libraries(ShapeAlign soft ply util ${FFTW_LIBRARIES})
target_Include_directories(ShapeAlign PUBLIC Include/ ${FFTW_INCLUDE_DIRS})

#-------------------------------------------------------------------------------
# Build ShapeDescriptor
#-------------------------------------------------------------------------------
add_executable(ShapeDescriptor ShapeDescriptor/ShapeDescriptor.cpp)
target_link_libraries(ShapeDescriptor soft ply util ${FFTW_LIBRARIES})
target_Include_directories(ShapeDescriptor PUBLIC Include/ ${FFTW_INCLUDE_DIRS})

#-------------------------------------------------------------------------------
# Build ShapeSymmetry
#-------------------------------------------------------------------------------
add_executable(ShapeSymmetry ShapeSymmetry/ShapeSymmetry.cpp)
target_link_libraries(ShapeSymmetry soft ply util ${FFTW_LIBRARIES})
target_Include_directories(ShapeSymmetry PUBLIC Include/ ${FFTW_INCLUDE_DIRS})