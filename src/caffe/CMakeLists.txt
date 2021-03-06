project( CaffeSrc )

#    Threads
find_package(Threads REQUIRED) 

#	Google-glog
find_package(Glog REQUIRED)
include_directories(${GLOG_INCLUDE_DIRS})

#    CUDA
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}
		-gencode arch=compute_35,code=sm_35
)

#    BLAS
if(BLAS STREQUAL "atlas")

    find_package(Atlas REQUIRED)
    include_directories(${ATLAS_INCLUDE_DIR})
    set(BLAS_LIBRARIES ${ATLAS_LIBRARIES})
    
elseif(BLAS STREQUAL "open")

    find_package(OpenBLAS REQUIRED)
    include_directories(${OpenBLAS_INCLUDE_DIR})
    set(BLAS_LIBRARIES ${OpenBLAS_LIB})
    
elseif(BLAS STREQUAL "mkl")

    find_package(MKL REQUIRED)
    include_directories(${MKL_INCLUDE_DIR})
    set(BLAS_LIBRARIES ${MKL_LIBRARIES})
    
endif()

#    HDF5
find_package(HDF5 COMPONENTS HL REQUIRED)
include_directories(${HDF5_INCLUDE_DIRS})

#    OpenCV
find_package(OpenCV COMPONENTS core highgui imgproc REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

#    LevelDB
find_package(LevelDB REQUIRED)
include_directories(${LEVELDB_INCLUDE})

#    Boost
find_package(Boost 1.49 COMPONENTS system REQUIRED)
include_directories( ${Boost_INCLUDE_DIR} )
link_directories( ${Boost_LIBRARY_DIRS} )

add_subdirectory(proto)

#    Recursively sources

#    cuda sources 
file(GLOB_RECURSE CU_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cu)

#    test sources
file(GLOB_RECURSE TEST_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/test_*.cpp)

#    all cpp sources
file(GLOB_RECURSE CPP_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)

#    remove test sources from cpp sources
list(REMOVE_ITEM CPP_SOURCES ${TEST_SOURCES})

add_library(caffe ${CPP_SOURCES})
cuda_add_library(caffe_cu ${CU_SOURCES})

# both depend on proto
add_dependencies(caffe proto)
add_dependencies(caffe_cu proto)

target_link_libraries(caffe caffe_cu proto
        ${GLOG_LIBRARIES}
        ${CUDA_curand_LIBRARY}
        ${HDF5_LIBRARIES} 
        ${OpenCV_LIBS} 
        ${LEVELDB_LIBS} 
        ${CUDA_CUBLAS_LIBRARIES}
        ${BLAS_LIBRARIES}
        ${CMAKE_THREAD_LIBS_INIT}
)

#set output directory
set_target_properties(caffe PROPERTIES 
        ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
        LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
)   

add_subdirectory(test)

###    Install    #################################################################################

install(TARGETS caffe DESTINATION lib)
