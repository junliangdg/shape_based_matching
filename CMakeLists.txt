cmake_minimum_required(VERSION 3.1)
project(shape_based_matching)

# arm or x86
IF(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    SET(PLATFORM_COMPILE_FLAGS "-mfpu=neon")
ELSE()
    SET(PLATFORM_COMPILE_FLAGS "-march=native")

    # some places of the algorithm are designed for 128 SIMD
    # so 128 SSE may slightly faster than 256 AVX, you may want this
#    SET(PLATFORM_COMPILE_FLAGS "-msse ")  # SSE only -msse2 -msse3 -msse4 -mssse3
ENDIF()

SET(COMMON_COMPILE_FLAGS "-fopenmp") # -fno-strict-aliasing
SET(CMAKE_CXX_FLAGS "${PLATFORM_COMPILE_FLAGS} ${COMMON_COMPILE_FLAGS} $ENV{CXXFLAGS}")
SET(CMAKE_CXX_FLAGS_DEBUG "-O0 -g2 -ggdb")
SET(CMAKE_CXX_FLAGS_RELEASE "-O3")

# Find OpenCV, you may need to set OpenCV_DIR variable
# to the absolute path to the directory containing OpenCVConfig.cmake file
# via the command line or GUI
find_package(OpenCV REQUIRED)

# If the package has been found, several variables will
# be set, you can find the full list with descriptions
# in the OpenCVConfig.cmake file.
# Print some message showing some of them
message(STATUS "OpenCV library status:")
message(STATUS "    config: ${OpenCV_DIR}")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

# include MIPP headers
include_directories (${INCLUDE_DIRECTORIES} "${CMAKE_CURRENT_SOURCE_DIR}/MIPP/")

# test exe
add_executable(${PROJECT_NAME}_test line2Dup.cpp test.cpp)
target_link_libraries(${PROJECT_NAME}_test ${OpenCV_LIBS})

# test exe
add_executable(${PROJECT_NAME}_test_fusion line2Dup.cpp test_fusion.cpp)
target_link_libraries(${PROJECT_NAME}_test_fusion ${OpenCV_LIBS})


find_package(OpenMP)
if(OpenMP_CXX_FOUND)
    target_link_libraries(${PROJECT_NAME}_test OpenMP::OpenMP_CXX)
    add_compile_definitions(_OPENMP)
endif()
