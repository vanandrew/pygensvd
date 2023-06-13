# Set minimum cmake version
cmake_minimum_required(VERSION 3.26)

# set c standard to c17
set(CMAKE_C_STANDARD 17)

# set project name and languages
project(omni LANGUAGES C)

# find python 3.10 > 3.7
find_package(Python 3.11 REQUIRED NumPy)


# find LAPACK and BLAS (Use static linkage)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR})
set(BLA_STATIC ON)
find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)
set(LAPACKE_STATIC TRUE)
find_package(LAPACKE REQUIRED)

# add pygensvd library
Python_add_library(
    _gsvd
    MODULE
    WITH_SOABI
    src/_gsvd.c
)
target_link_libraries(
    _gsvd
    PRIVATE
    Python::NumPy
)

# check if this is an mkl library
if(BLAS_LIBRARIES MATCHES "(mkl)")
    if(NOT DEFINED ENV{MKLROOT})
        message(
            FATAL_ERROR
            "CMake detected MKL for a BLAS/LAPACK backend, but MKLROOT is undefined! \
            Check that your MKL environment is setup correctly."
        )
    endif()

    set(MKLROOT $ENV{MKLROOT})
    set(MKL_INCLUDE_DIRECTORY ${MKLROOT}/include)
    target_compile_definitions(
        _gsvd
        PRIVATE
        USE_MKL
    )
    target_include_directories(
        _gsvd
        PRIVATE
        ${MKL_INCLUDE_DIRECTORY}
    )
else()
    target_compile_definitions(
        _gsvd
        PRIVATE
        USE_LAPACK
    )
endif()
target_include_directories(
    _gsvd
    PRIVATE
    ${LAPACKE_INCLUDE_DIRS_DEP}
)
target_link_libraries(
    _gsvd
    PRIVATE
    ${LAPACKE_LIBRARIES_DEP}
)

# install library
install(
    TARGETS _gsvd
    LIBRARY DESTINATION lib
)
