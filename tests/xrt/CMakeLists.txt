CMAKE_MINIMUM_REQUIRED(VERSION 3.0.0)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_VERBOSE_MAKEFILE ON)
set(XILINX_XRT $ENV{XILINX_XRT})
set(XRT_CORE_LIBRARY xrt_core)
set(MODE hw)

if (DEFINED ENV{XCL_EMULATION_MODE})
  set(MODE $ENV{XCL_EMULATION_MODE})
  string(REPLACE "_" "" XCL_EMU_SUFFIX ${MODE})
  set(XRT_CORE_LIBRARY xrt_${XCL_EMU_SUFFIX})
  message("=============== XRT_CORE_LIB=${XRT_CORE_LIBRARY}")
endif()

find_library(xrt_core_LIBRARY
  NAMES ${XRT_CORE_LIBRARY}
  HINTS "${XILINX_XRT}/lib")
message("xrt_core_LIBRARY=${xrt_core_LIBRARY}")

find_library(xrt_coreutil_LIBRARY
  NAMES xrt_coreutil
  HINTS "${XILINX_XRT}/lib")
message("xrt_coreutil_LIBRARY=${xrt_coreutil_LIBRARY}")

find_library(xrt_xilinxopencl_LIBRARY
  NAMES xilinxopencl
  HINTS "${XILINX_XRT}/lib")
message("xrt_xilinxopencl_LIBRARY=${xrt_xilinxopencl_LIBRARY}")

if (NOT WIN32)
find_library(uuid_LIBRARY
  NAMES uuid)
message("uuid_LIBRARY=${uuid_LIBRARY}")
endif (NOT WIN32)

include_directories(${XILINX_XRT}/include)

if (NOT DEFINED ${INSTALL_DIR})
  set(INSTALL_DIR "${CMAKE_CURRENT_SOURCE_DIR}/build/${CMAKE_SYSTEM_NAME}/${CMAKE_BUILD_TYPE}/${MODE}")
endif()

add_subdirectory(00_hello)
add_subdirectory(02_simple)
add_subdirectory(03_loopback)
add_subdirectory(04_swizzle)
add_subdirectory(07_sequence)
add_subdirectory(11_fp_mmult256)
add_subdirectory(13_add_one)
add_subdirectory(22_verify)
add_subdirectory(56_xclbin)
add_subdirectory(100_ert_ncu)
if (NOT WIN32)
  add_subdirectory(102_multiproc_verify)
endif(NOT WIN32)
