cmake_minimum_required(VERSION 3.18)

if(NOT ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
	message(FATAL "This library only for Apple operating systems. Current OS: ${CMAKE_SYSTEM_NAME}")
endif()

set(CMAKE_CXX_STANDARD 17)

# Specify Metal version to fetch and include
# AUTO - get the latest stable release
# AUTO BETA - get the latest beta release
# MAC12..MAC15 [BETA] fetch the latest metal version for specific MacOS version (or BETA)
# IOS15..IOS18 [BETA] fetch the latest metal version for specific iOS version (or BETA)
# Specify exact version
# V_120_150 metal-cpp_macOS12_iOS15
# V_130_160 metal-cpp_macOS13_iOS16
# V_133_164 metal-cpp_macOS13.3_iOS16.4
# V_140_170_B metal-cpp_macOS14_iOS17-beta
# V_142_172 metal-cpp_macOS14.2_iOS17.2
# V_150_180_B metal-cpp_macOS15_iOS18-beta
set(METAL_VERSION "" CACHE STRING "Metal version to include")
string(TOUPPER "${METAL_VERSION}" METAL_VERSION_V)

if("${METAL_VERSION_V}" STREQUAL "")
	set(METAL_VERSION_V "AUTO")
endif()

if("${METAL_VERSION_V}" MATCHES ".+$BETA")
	set(METAL_BETA ON)
endif()

if("${METAL_VERSION_V}" STREQUAL "AUTO")
	# determine version from current host MacOS version
elseif("${METAL_VERSION_V}" STREQUAL "V_120_150")
elseif("${METAL_VERSION_V}" STREQUAL "V_130_160")
elseif("${METAL_VERSION_V}" STREQUAL "V_133_164")
elseif("${METAL_VERSION_V}" STREQUAL "V_140_170_B")
elseif("${METAL_VERSION_V}" STREQUAL "V_142_172")
	set(METAL_CPP_ARCHIVE_FILE "https://developer.apple.com/metal/cpp/files/metal-cpp_macOS14.2_iOS17.2.zip")
elseif("${METAL_VERSION_V}" STREQUAL "V_150_180_B")
else()
	message(FATAL_ERROR "Unsupported metal-cpp version: ${METAL_VERSION_V}")
endif()

message(STATUS "Ready to download ${METAL_CPP_ARCHIVE_FILE}")

set(METAL_FETCHED "metal-cpp")

if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/${METAL_FETCHED}.zip")
	file(DOWNLOAD ${METAL_CPP_ARCHIVE_FILE} "${CMAKE_CURRENT_BINARY_DIR}/${METAL_FETCHED}.zip" SHOW_PROGRESS)
endif()

if(EXISTS "${CMAKE_CURRENT_BINARY_DIR}/${METAL_FETCHED}.zip")
	file(ARCHIVE_EXTRACT INPUT "${CMAKE_CURRENT_BINARY_DIR}/${METAL_FETCHED}.zip" DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/${METAL_FETCHED}")
else()
	message(FATAL_ERROR "No metal-cpp file!")
endif()

# Library definition
add_library(MetalCpp INTERFACE)

# Metal cpp headers
target_include_directories(MetalCpp
	INTERFACE
		"${CMAKE_CURRENT_BINARY_DIR}/${METAL_FETCHED}/metal-cpp"
		metal-cpp-extensions
)

# Metal cpp library (linker)
target_link_libraries(MetalCpp
	INTERFACE
		"-framework Metal"
		"-framework MetalKit"
		"-framework AppKit"
		"-framework Foundation"
		"-framework QuartzCore"
)
