cmake_minimum_required(VERSION 3.10)

include(cmake/cable/bootstrap.cmake)

include(CableBuildInfo)
include(CableBuildType)
include(CableToolchains)
include(HunterGate)

cable_configure_toolchain(DEFAULT cxx17)
cable_set_build_type(DEFAULT Release CONFIGURATION_TYPES Release Debug RelWithDebInfo)

set(HUNTER_CONFIGURATION_TYPES Release CACHE STRING "Build type of Hunter packages")
set(HUNTER_JOBS_NUMBER 6 CACHE STRING "Number of parallel builds used by Hunter")
set(HUNTER_USE_CACHE_SERVERS NO)
HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.23.267.tar.gz"
  SHA1 "9c5c7fa6e17c2ae15dd922184bc7c51235aaae70"
  LOCAL
)


project(evrprogpowminer)
set(PROJECT_VERSION 1.3.0)


option(ETHASHCL "Build with OpenCL mining" ON)
option(ETHASHCUDA "Build with CUDA mining" ON)
option(ETHASHCPU "Build with CPU mining (only for development)" OFF)
option(ETHDBUS "Build with D-Bus support" OFF)
option(APICORE "Build with API Server support" ON)
option(DEVBUILD "Log developer metrics" OFF)

# propagates CMake configuration options to the compiler
function(configureProject)
	if (ETHASHCL)
		add_definitions(-DETH_ETHASHCL)
	endif()
	if (ETHASHCUDA)
		add_definitions(-DETH_ETHASHCUDA)
	endif()
	if (ETHASHCPU)
		add_definitions(-DETH_ETHASHCPU)
	endif()
	if (ETHDBUS)
		add_definitions(-DETH_DBUS)
	endif()
	if (APICORE)
		add_definitions(-DAPI_CORE)
	endif()
	if (DEVBUILD)
		add_definitions(-DDEV_BUILD)
	endif()
endfunction()

hunter_add_package(Boost COMPONENTS system filesystem thread)
hunter_add_package(jsoncpp)
hunter_add_package(intx)

find_package(Boost CONFIG REQUIRED COMPONENTS system filesystem thread)
find_package(jsoncpp CONFIG REQUIRED)
find_package(intx CONFIG REQUIRED)

configureProject()

message("----------------------------------------------------------------------------")
message("-- CMake ${CMAKE_VERSION}")
message("-- Build ${CMAKE_BUILD_TYPE} / ${CMAKE_SYSTEM_NAME}")
message("----------------------------------------------------------------- components")
message("-- ETHASHCL         Build OpenCL components                      ${ETHASHCL}")
message("-- ETHASHCUDA       Build CUDA components                        ${ETHASHCUDA}")
message("-- ETHASHCPU        Build CPU components (only for development)  ${ETHASHCPU}")
message("-- ETHDBUS          Build D-Bus components                       ${ETHDBUS}")
message("-- APICORE          Build API Server components                  ${APICORE}")
message("-- DEVBUILD         Build with dev logging                       ${DEVBUILD}")
message("----------------------------------------------------------------------------")
message("")

include(EthCompilerSettings)
if(UNIX AND NOT APPLE)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static-libstdc++")
endif()


cable_add_buildinfo_library(PROJECT_NAME ${PROJECT_NAME})

add_subdirectory(libcrypto)
add_subdirectory(libdevcore)
add_subdirectory(libethcore)
add_subdirectory(libhwmon)
add_subdirectory(libpoolprotocols)

if (ETHASHCL)
	add_subdirectory(libethash-cl)
endif ()
if (ETHASHCUDA)
	add_subdirectory(libethash-cuda)
endif ()
if (ETHASHCPU)
	add_subdirectory(libethash-cpu)
endif ()
if (APICORE)
	add_subdirectory(libapicore)
endif()

add_subdirectory(evrprogpowminer)


if(WIN32)
	set(CPACK_GENERATOR ZIP)
else()
	set(CPACK_GENERATOR TGZ)
endif()
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_CHECKSUM SHA256)
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
include(CPack)
