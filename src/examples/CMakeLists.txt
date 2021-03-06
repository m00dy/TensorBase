

cmake_minimum_required(VERSION 3.8.2 FATAL_ERROR)
project("TensorBase_class_examples_tensorbase")

message(STATUS "building examples...")

#------------------------------------------------------------------------------
# get the test executables
include(executables.cmake)

#------------------------------------------------------------------------------
# Include directories for examples
set(SMARTPEAK_EXAMPLES_INTERNAL_INCLUDE_DIRECTORIES "${PROJECT_BINARY_DIR}/include/")
# add TensorBase directories
set(SMARTPEAK_EXAMPLES_EXTERNAL_INCLUDE_DIRECTORIES "${TensorBase_INCLUDE_DIRECTORIES}")
include_directories(${SMARTPEAK_EXAMPLES_INTERNAL_INCLUDE_DIRECTORIES})
include_directories(SYSTEM ${SMARTPEAK_EXAMPLES_EXTERNAL_INCLUDE_DIRECTORIES})

#------------------------------------------------------------------------------
# disable optimization for examples for gcc like compilers
if (CMAKE_COMPILER_IS_INTELCXX OR CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CLANG)
  set(_TMP_CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})
  set(CMAKE_CXX_FLAGS_RELEASE "-O0")
endif()

#------------------------------------------------------------------------------
# # core examples
# add_custom_target(CORE_EXAMPLES)
# add_dependencies(CORE_EXAMPLES ${core_executables_list})

# cuda examples
if (EVONET_CUDA)
  add_custom_target(CUDA_EXAMPLES)
  add_dependencies(CUDA_EXAMPLES ${cuda_executables_list})
endif()

#------------------------------------------------------------------------------
# Add the actual examples
# foreach(_examples ${core_executables_list})
  # add_executable(${_examples} source/${_examples})
  # target_link_libraries(${_examples} ${TensorBase_LIBRARIES})
  # # only add OPENMP flags to gcc linker (execpt Mac OS X, due to compiler bug
  # # see https://sourceforge.net/apps/trac/open-ms/ticket/280 for details)
  # if (OPENMP_FOUND AND NOT MSVC AND NOT ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    # set_target_properties(${_examples} PROPERTIES LINK_FLAGS ${OpenMP_CXX_FLAGS})
  # endif()
# endforeach(_examples)

if (EVONET_CUDA)
  foreach(_examples ${cuda_executables_list})
	add_executable(${_examples} source/${_examples})
	target_link_libraries(${_examples} ${TensorBase_LIBRARIES})
	# only add OPENMP flags to gcc linker (execpt Mac OS X, due to compiler bug
	# see https://sourceforge.net/apps/trac/open-ms/ticket/280 for details)
	if (OPENMP_FOUND AND NOT MSVC AND NOT ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	  set_target_properties(${_examples} PROPERTIES LINK_FLAGS ${OpenMP_CXX_FLAGS})
	endif()
	target_link_libraries(${_examples} ${CUDA_LIBRARIES})
	set_property(TARGET ${_examples} PROPERTY CUDA_STANDARD 11)
  endforeach(_examples)
endif()
#------------------------------------------------------------------------------
# restore old compiler flags
if (CMAKE_COMPILER_IS_INTELCXX OR CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CLANG)
  set(${CMAKE_CXX_FLAGS_RELEASE} ${_TMP_CMAKE_CXX_FLAGS_RELEASE})
endif()

#------------------------------------------------------------------------------
# add filenames to Visual Studio solution tree
set(sources_VS)
foreach(i ${EXAMPLE_executables})
  list(APPEND sources_VS "${i}")
endforeach(i)
source_group("" FILES ${sources_VS})