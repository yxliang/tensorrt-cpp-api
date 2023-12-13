cmake_minimum_required(VERSION 3.18)
project(tensorrt_cpp_api)

# Use ccache to speed up rebuilds
include(cmake/ccache.cmake)

# Set C++ version and optimization level
set(CMAKE_CXX_STANDARD 17)
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Ofast -DNDEBUG -Wno-deprecated-declarations")
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_DEBUG_POSTFIX "d")
SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON) 

# For finding FindTensorRT.cmake
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

SET(3rdlib_DIR "D:/Libraries/")
SET(TensorRT_DIR "${3rdlib_DIR}/TensorRT")
SET(OpenCV_DIR "${3rdlib_DIR}/OpenCV-cuda")
# TODO: Specify the path to TensorRT root dir
if (NOT TensorRT_DIR)
    set(TensorRT_DIR /home/cyrus/work/libs/TensorRT-8.6.1.6/)
endif()
# Use the correct version of CUDA

IF(UNIX)
	set(CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda)
ENDIF()

# We require CUDA, OpenCV, and TensorRT
find_package(TensorRT REQUIRED)
find_package(CUDA REQUIRED)
find_package(OpenCV REQUIRED)

add_library(tensorrt_cpp_api SHARED
        src/engine.cpp)
IF(MSVC)		
    INCLUDE(GenerateExportHeader)
    GENERATE_EXPORT_HEADER(tensorrt_cpp_api
        BASE_NAME TRT
        # EXPORT_MACRO_NAME Chromosome_EXPORTS
        EXPORT_FILE_NAME ${CMAKE_SOURCE_DIR}/src/tensorrt_cpp_api_export.h
    )
ENDIF(MSVC)

target_include_directories(tensorrt_cpp_api PUBLIC ${OpenCV_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS} ${TensorRT_INCLUDE_DIRS})
target_link_libraries(tensorrt_cpp_api PUBLIC ${OpenCV_LIBS} ${CUDA_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${TensorRT_LIBRARIES})
SET_TARGET_PROPERTIES(tensorrt_cpp_api PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})

add_executable(run_inference_benchmark src/main.cpp)
target_link_libraries(run_inference_benchmark tensorrt_cpp_api)
SET_TARGET_PROPERTIES(run_inference_benchmark PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})