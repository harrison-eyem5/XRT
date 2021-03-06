include_directories(
  ${CMAKE_BINARY_DIR} # includes version.h
  )

if (NOT WIN32)
include_directories(${DRM_INCLUDE_DIRS})
else()
set(Boost_USE_STATIC_LIBS     ON)
set(Boost_USE_MULTITHREADED   ON)
find_package(Boost REQUIRED COMPONENTS system filesystem )
include_directories(${Boost_INCLUDE_DIRS})
endif (NOT WIN32)

file(GLOB XRT_CORE_COMMON_LIB_FILES
  "config_reader.cpp"
  "debug.cpp"
  "device.cpp"
  "error.cpp"
  "message.cpp"
  "time.cpp"
  "xclbin_parser.cpp"
  "sensor.cpp"
  "system.cpp"
  "utils.cpp"
  "thread.cpp"
  "module_loader.cpp"
  "api/*.cpp"
  )

if (DEFINED XRT_AIE_BUILD)
  file(GLOB XRT_CORE_COMMON_AIE_LIB_FILES
    "api/aie/*.cpp"
    )
  list(APPEND XRT_CORE_COMMON_LIB_FILES ${XRT_CORE_COMMON_AIE_LIB_FILES})
endif()

# Files to include in object list
file(GLOB XRT_CORE_COMMON_OBJ_FILES
  "scheduler.*"
  )

add_library(xrt_coreutil SHARED ${XRT_CORE_COMMON_LIB_FILES})
add_library(xrt_coreutil_static STATIC ${XRT_CORE_COMMON_LIB_FILES})

set_target_properties(xrt_coreutil PROPERTIES VERSION ${XRT_VERSION_STRING}
  SOVERSION ${XRT_SOVERSION})

message("bfl=${Boost_FILESYSTEM_LIBRARY}")

target_link_libraries(xrt_coreutil
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  )

if (NOT WIN32)
  target_link_libraries(xrt_coreutil uuid pthread dl)
endif()

install(TARGETS xrt_coreutil
  LIBRARY DESTINATION ${XRT_INSTALL_LIB_DIR} ${XRT_NAMELINK_SKIP}
  RUNTIME DESTINATION ${XRT_INSTALL_BIN_DIR}
)

install(TARGETS xrt_coreutil
  ARCHIVE DESTINATION ${XRT_INSTALL_LIB_DIR} COMPONENT ${XRT_DEV_COMPONENT}
  LIBRARY DESTINATION ${XRT_INSTALL_LIB_DIR} COMPONENT ${XRT_DEV_COMPONENT} ${XRT_NAMELINK_ONLY}
)

# The scheduler object files are for auto config of scheduler. These
# files reference xrt_core symbols, hence are excluded from
# xrt_corecommon shared library and instead linked explicitly into
# client (core) libraries
add_library(core_common_objects OBJECT ${XRT_CORE_COMMON_OBJ_FILES})
