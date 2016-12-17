cmake_minimum_required(VERSION 2.8)

project(kfusion)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_FORCE_INLINES")

find_package(CUDA REQUIRED)
find_package(TooN REQUIRED)
find_package(OpenGL)
find_package(Freenect)
find_package(GLUT)


include_directories(
	${CMAKE_BINARY_DIR}/include
	${TOON_INCLUDE_PATHS}
)

cuda_add_executable(devicequery
	devicequery.cpp
	)
target_link_libraries(devicequery cuda)

set(kfusion_cuda_srcs
	helpers.cu
	helpers.h
	kfusion.cu
	kfusion.h
	cutil_math.h
	)
	
cuda_add_library(kfusion STATIC
	${kfusion_cuda_srcs}
	perfstats.h
	OPTIONS
	-O3
	)
target_link_libraries(kfusion cuda)

if (GLUT_FOUND)

	include_directories(${GLUT_INCLUDE_DIR})

	add_executable(kfusion_test
		test.cpp
		)
	
	target_link_libraries(kfusion_test kfusion ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES}) 
	
    if(FREENECT_FOUND)
        include_directories(${FREENECT_INCLUDE_PATHS})
        add_definitions(-DLIBFREENECT_INTERFACE)
        add_executable(kfusion_kinect
            kinect.cpp interface.h interface.cpp
            )
        target_link_libraries(kfusion_kinect kfusion ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES} ${FREENECT_LIBRARIES}) 
    else()
        include_directories($ENV{KINECTSDK10_DIR}/inc)
        add_definitions(-DMS_KINECT_INTERFACE)
        add_executable(kfusion_kinect
            kinect.cpp interface.h interface.cpp
            )
        target_link_libraries(kfusion_kinect kfusion ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES} $ENV{KINECTSDK10_DIR}/lib/amd64/Kinect10.lib) 
    endif()

endif()
