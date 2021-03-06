##===------------------------------------------------------------------------------*- CMake -*-===##
##
##                                   S E R I A L B O X
##
## This file is distributed under terms of BSD license. 
## See LICENSE.txt for more information.
##
##===------------------------------------------------------------------------------------------===##
##
## This is an example CMake file which shows a standalone Fortran example.
##
##===------------------------------------------------------------------------------------------===##

cmake_minimum_required(VERSION 3.12)

#
# First, we setup the project and look for Serialbox. This is a pure Fortran project, however we 
# still require the C++ compiler for linking (i.e we depend on libstdc++ or libc++).
#
project(SerialboxPerturbationExample CXX Fortran)

#
# Now, we call the find_package-module and specifically request the Fortran interface. Note that 
# this obviously required you to compile Serialbox with SERIALBOX_FORTRAN_ENABLE=ON.
#
find_package(Serialbox REQUIRED COMPONENTS Fortran)

if(NOT SERIALBOX_HAS_NETCDF)
    message(FATAL_ERROR "Serialbox was not compiled with netCDF support, cannot compile this example.")
endif()

#
# Set some Fortran specific compiler flags (e.g enable preprocessing) and set the corresponding 
# flags for serialization.
#
add_definitions(-DSERIALIZE)

#
# Here we preprocess all the source files, using the preprocesser script pp_ser.py, and store them 
# in ${PROJECT_SOURCE_DIR}/pp.

set(PP_OUTPUT ${PROJECT_SOURCE_DIR}/pp)
serialbox_run_pp_ser(SOURCES main_producer.F90  
                             main_consumer.F90
                             main_consumer_perturb.F90
                             m_ser.F90 
                     OUTPUT_DIR ${PP_OUTPUT})

# The source files of `add_executable`/`add_library` are the preprocessed files from before.
add_library(m_ser OBJECT ${PP_OUTPUT}/m_ser.F90)
target_link_libraries(m_ser Serialbox::SerialboxFortranStatic)

## Producer
add_executable(fortran_producer_netcdf ${PP_OUTPUT}/main_producer.F90)
target_link_libraries(fortran_producer_netcdf m_ser)
set_target_properties(fortran_producer_netcdf PROPERTIES LINKER_LANGUAGE Fortran)

## Consumer
add_executable(fortran_consumer_netcdf ${PP_OUTPUT}/main_consumer.F90)
target_link_libraries(fortran_consumer_netcdf m_ser)
set_target_properties(fortran_consumer_netcdf PROPERTIES LINKER_LANGUAGE Fortran)

## Perturbed Consumer
add_executable(fortran_consumer_perturb_netcdf ${PP_OUTPUT}/main_consumer_perturb.F90)
target_link_libraries(fortran_consumer_perturb_netcdf m_ser)
set_target_properties(fortran_consumer_perturb_netcdf PROPERTIES LINKER_LANGUAGE Fortran)

#
# Finally, copy our run script to the build directory
#
file(COPY ${PROJECT_SOURCE_DIR}/run.sh DESTINATION ${PROJECT_BINARY_DIR})
