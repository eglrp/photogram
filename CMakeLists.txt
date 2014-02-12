cmake_minimum_required (VERSION 2.8)
project (photogram CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

### Sets ###
option(DEBUG "Debug output"	TRUE)
option(USE_SIFT_GPU "Use SiftGPU to compute image features" TRUE)


include(FindPkgConfig)
include(ExternalProject)

### dependencies ###
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules/" ${CMAKE_MODULE_PATH})

## Debug ##
if(DEBUG)
	message(STATUS "Debug on")
else(DEBUG)
	message(STATUS "Debug off")
	add_definitions(-DNDEBUG=1)
endif(DEBUG)

##############
### OpenCV ###
##############
find_package(OpenCV REQUIRED)
set(LINKER_LIBS ${LINKER_LIBS} ${OpenCV_LIBS})

################
### SIFT Gpu ###
################
# Note (mtourne): eventually use CudaSIFT directly :
# https://github.com/Celebrandil/CudaSift
set(ENV{SIFT_GPU_MODE}		2) #CUDA = 1, GLSL = 2
set(ENV{SIFT_GPU_CUDA_PATH}	/usr/local/cuda)

if (USE_SIFT_GPU)
	message(STATUS "Using SiftGPU")
	find_package(OpenGL REQUIRED)
	find_package(GLUT REQUIRED)
	find_package(Glew)

	ExternalProject_Add(SIFTGPU
                    SOURCE_DIR ${CMAKE_SOURCE_DIR}/SiftGPU
					CMAKE_ARGS
					   -DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}
                       -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}
                       -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> )

	ExternalProject_Get_Property(SIFTGPU install_dir)
	include_directories(${install_dir}/include)
	set(LINKER_LIBS ${LINKER_LIBS} ${install_dir}/lib/libsiftgpu.a)
	set(LINKER_LIBS ${LINKER_LIBS} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES})
# XX automatically include that from SiftGPU/src/SiftGPU/CMakeLists.txt ?

	add_definitions(-DUSE_SIFT_GPU)
	add_definitions(-DSIFT_GPU_MODE=$ENV{SIFT_GPU_MODE})

	ExternalProject_Get_Property(SIFTGPU )
else (USE_SIFT_GPU)
	message(STATUS "Using OpenCV Sift")
endif (USE_SIFT_GPU)


#############
### Eigen ###
#############
#find_package(Eigen REQUIRED)
#include_directories(SYSTEM ${EIGEN_INCLUDE_DIRS})

#############
### Boost ###
#############
find_package(Boost COMPONENTS system REQUIRED)
set(LINKER_LIBS ${LINKER_LIBS} ${Boost_SYSTEM_LIBRARY})

############
### MISC ###
############
# include rapidjson lib
include_directories(rapidjson/include)


###############
### Targets ###
###############
# CXX Flags
add_definitions(-std=c++11)
add_definitions(-W -Wall)

### sources ###
add_executable(photogram
	photogram.cc
	features2d.cc
	image.cc
	image_pairs.cc
	bundle.cc
	easyexif/exif.cpp
	sift_gpu_wrapper.cpp
	util.cc
)
target_link_libraries(photogram ${LINKER_LIBS})

add_executable(homography
	homography.cc
	features2d.cc
	image.cc
	sift_gpu_wrapper.cpp
	util.cc
)
target_link_libraries(homography ${LINKER_LIBS})

add_executable(test_haversine_flann
	test_haversine_flann.cc
)
target_link_libraries(test_haversine_flann ${LINKER_LIBS})