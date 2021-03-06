#  Cinder -- C++ Single Spiking Neuron Simulator
#  Copyright (C) 2015, 2016  Andreas Stöckel, Christoph Jenzen
#
#  This program is free software: you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation, either version 3 of the License, or
#  (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with this program.  If not, see <http://www.gnu.org/licenses/>.

# CMake version and project metadata
cmake_minimum_required(VERSION 3.2)
project(cinder)

# Add the local "cmake" directory to the search path
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake)

# Automatically download and compile Google-Test and the JSON library
include(gtest_external)

# Enable C++14 support
set(CMAKE_CXX_STANDARD 14)

# Enable all warnings on MSVC and GCC/Clang/Intel
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
		string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
	else()
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
	endif()
elseif(("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU") OR
		("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang") OR
		("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel"))
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic -march=native -ffast-math -fno-finite-math-only")
	if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
		# Clang Bug 21629 https://llvm.org/bugs/show_bug.cgi?id=21629
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-missing-braces")
		# Disable annoying -Wunused-const-variable warning in conjunction with
		# the NAMED_VECTOR_ELEMENT/TYPED_VECTOR_ELEMENT macros
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-const-variable")
	endif ()
endif()

# Set a default build type
if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif()

include_directories(${CMAKE_SOURCE_DIR} ${CMAKE_BINARY_DIR})

# Add configuration option for setting the width of the floating point values
set(CINDER_REAL_WIDTH 8 CACHE STRING "Width of the floating point values in bytes (one of 4, 8, 10, 16)")
set(CINDER_USE_FAST_EXP 1 CACHE FLAG "If true, an approximation of the exponential function is used to improve performance for the AdEx model.")

configure_file(cinder/config.h.in cinder/config.h)

# Compile the library itself
add_library(cinder
	cinder/common/array_utils
	cinder/common/fast_math
	cinder/common/spikes
	cinder/common/time
	cinder/common/types
	cinder/common/vector
	cinder/models/current_source
	cinder/models/neuron
	cinder/models/noise_source
	cinder/models/synapse
	cinder/models/neurons/adex
	cinder/models/neurons/hodgkin_huxley
	cinder/models/neurons/izhikevich
	cinder/models/neurons/lif
	cinder/models/neurons/mat
	cinder/models/synapses/cond_alpha
	cinder/models/synapses/cond_exp
	cinder/models/synapses/cur_alpha
	cinder/models/synapses/cur_exp
	cinder/models/synapses/delta
	cinder/integrator/dormand_prince
	cinder/integrator/euler
	cinder/integrator/midpoint
	cinder/integrator/runge_kutta
	cinder/ode/controller
	cinder/ode/ode
	cinder/ode/recorder
	cinder/ode/solver
)

# Installation configuration
install(TARGETS cinder ARCHIVE DESTINATION lib)
install(
	DIRECTORY ${CMAKE_SOURCE_DIR}/cinder
	DESTINATION include
	FILES_MATCHING PATTERN "*.hpp"
)
install(
	DIRECTORY ${CMAKE_BINARY_DIR}/cinder
	DESTINATION include
	FILES_MATCHING PATTERN "*.h"
)
install(
	FILES ${CMAKE_SOURCE_DIR}/cmake/FindCinder.cmake
	DESTINATION share/cmake/Cinder/
	RENAME CinderConfig.cmake
)

# Submodules
enable_testing()
add_subdirectory(examples)
add_subdirectory(test)

