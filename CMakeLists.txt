#==================================================================================================
#
#	CMake file for building ozHAL
#
#==================================================================================================


#--------------------------------------------------------------------------------------------------
#
#	CMake compatibility issues
#
cmake_minimum_required( VERSION 2.8.8 )


#--------------------------------------------------------------------------------------------------
#
#	Project name
#
project( objectTracker )


#--------------------------------------------------------------------------------------------------
#
#   Module finding
#
set( CMAKE_MODULE_PATH  ${PROJECT_SOURCE_DIR}/cmake/modules )
find_package( LOKI			1.0.7	REQUIRED )
find_package( VITALS		0.4.0	REQUIRED )
find_package( ROBBIE		0.3.0	REQUIRED )
find_package( OpenCV		2.4.5	REQUIRED )


#--------------------------------------------------------------------------------------------------
#
#   Project options
#
option( INSTALL_DOC	"Set to ON to skip build/install Documentation"	OFF )


#--------------------------------------------------------------------------------------------------
#
#   Source & Include directories
#
set( INCLUDE_DIR	${PROJECT_SOURCE_DIR}/include )
set( SOURCE_DIR		${PROJECT_SOURCE_DIR}/src )


#--------------------------------------------------------------------------------------------------
#
#   Additional include directories
#
include_directories( ${INCLUDE_DIR} )


#--------------------------------------------------------------------------------------------------
#
#	Set sources to compile
#
set( EXECUTABLE_SOURCES
	${SOURCE_DIR}/EntryPoint.cpp
	${SOURCE_DIR}/ObjectTracker.cpp
	${SOURCE_DIR}/Finder.cpp
	${SOURCE_DIR}/Histogram.cpp
	${SOURCE_DIR}/SaladDetector.cpp
)


#--------------------------------------------------------------------------------------------------
#
#   Project version
#
file( READ "${INCLUDE_DIR}/BuildVersion.h" PROGRAM_VERSION )

string( REGEX MATCH "define[ \t]+PROGRAM_MAJOR_VERSION[ \t]+[\"]([0-9]+)+[\"]"
	PROGRAM_MAJOR_VERSION_MATCH "${PROGRAM_VERSION}" )
set( ${PROJECT_NAME}_MAJOR_VERSION "${CMAKE_MATCH_1}" )

string( REGEX MATCH "define[ \t]+PROGRAM_MINOR_VERSION[ \t]+[\"]([0-9]+)+[\"]"
	PROGRAM_MINOR_VERSION_MATCH "${PROGRAM_VERSION}" )
set( ${PROJECT_NAME}_MINOR_VERSION "${CMAKE_MATCH_1}" )

string( REGEX MATCH "define[ \t]+PROGRAM_PATCH_VERSION[ \t]+[\"]([0-9]+)+[\"]"
	PROGRAM_PATCH_VERSION_MATCH "${PROGRAM_VERSION}" )
set( ${PROJECT_NAME}_PATCH_VERSION "${CMAKE_MATCH_1}" )

set( ${PROJECT_NAME}_VERSION
	${${PROJECT_NAME}_MAJOR_VERSION}.${${PROJECT_NAME}_MINOR_VERSION}.${${PROJECT_NAME}_PATCH_VERSION} )


#--------------------------------------------------------------------------------------------------
#
#	Warning Options / C++ Language Options / Code Generation Options
#
add_definitions(
	-Wall
	-W
	-Wextra
	-pedantic
	-Winline
	# -Weffc++
	-std=c++1y
	-Wc++11-compat
	-Wconversion
	-Wconversion-null
	-Wsign-conversion
	-Wctor-dtor-privacy
	-Wdelete-non-virtual-dtor
	-Wnarrowing
	-Wnon-virtual-dtor
	-Wreorder
	-Wshadow
	-Wold-style-cast
	-Woverloaded-virtual
	-Wno-pmf-conversions
	-Wsign-promo
	-Wcast-align
	-Wfloat-equal
	-Wformat=2
	-Wmissing-declarations
	-Woverlength-strings
	-Wunreachable-code
	-fconstexpr-depth=10
	-fno-elide-constructors
	-ffor-scope
	-fno-gnu-keywords
	-ftemplate-backtrace-limit=10
	-fuse-cxa-atexit
	-fexceptions
	-fsigned-char
	-fdiagnostics-color
	-fPIE
)


#--------------------------------------------------------------------------------------------------
#
# Set default cmake build type to Debug (None Debug Release RelWithDebInfo)
#
if( NOT CMAKE_BUILD_TYPE )
	set( CMAKE_BUILD_TYPE "Debug" )
endif()


if( CMAKE_BUILD_TYPE STREQUAL "Debug" )
	add_definitions(
		-fno-optimize-sibling-calls
        -fno-omit-frame-pointer

		-fsanitize=address
		-fsanitize=undefined
		#-fsanitize=thread
	)
endif()


#--------------------------------------------------------------------------------------------------
#
#   Add builds definitions
#
if( CMAKE_BUILD_TYPE STREQUAL "Debug" )
	add_definitions( -O1 -DDEBUG -g )
elseif( CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo" )
	add_definitions( -O3 -DDEBUG -march=native -msse -msse2 -msse3 -mssse3 -msse4.1 -msse4.2 )
elseif( CMAKE_BUILD_TYPE STREQUAL "Release" )
	add_definitions( -O3 -march=native -msse -msse2 -mssse3 -msse4.1 -msse4.2 -ffast-math -ftree-loop-if-convert -funroll-loops -mfpmath=sse )
endif()

#--------------------------------------------------------------------------------------------------
#
#   Executable creation
#
add_executable( ${PROJECT_NAME} ${EXECUTABLE_SOURCES} )


#--------------------------------------------------------------------------------------------------
#
#   Library includes
#
target_include_directories( ${PROJECT_NAME} SYSTEM PUBLIC
	${OpenCV_INCLUDE_DIRS}
	${VITALS_INCLUDE_DIRS}
	${ROBBIE_INCLUDE_DIRS}
)


#--------------------------------------------------------------------------------------------------
#
#   Executable linking
#
target_link_libraries( ${PROJECT_NAME}
	-pie

	${OpenCV_LIBRARIES}
	${VITALS_LIBRARIES}
	${ROBBIE_LIBRARIES}
)

if( CMAKE_BUILD_TYPE STREQUAL "Debug" )
	target_link_libraries( ${PROJECT_NAME}
			-lasan
			-lubsan
			#-ltsan
		)
endif()

#--------------------------------------------------------------------------------------------------
#
#   Executable properties
#
set_target_properties( ${PROJECT_NAME} PROPERTIES

	# allow creating static and shared libs without conflicts
	CLEAN_DIRECT_OUTPUT 1

	# Executable name
	OUTPUT_NAME ${PROJECT_NAME}-${${PROJECT_NAME}_VERSION}
)


#--------------------------------------------------------------------------------------------------
#
#   Copying needed files
#
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE} )
file( COPY ${PROJECT_SOURCE_DIR}/resources/ DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} )


#--------------------------------------------------------------------------------------------------
#
#   Display status message for important variables
#
message( STATUS )
message( STATUS "-------------------------------------------------------------------------------" )
message( STATUS "CMAKE_BUILD_TYPE = ${CMAKE_BUILD_TYPE}" )
message( STATUS "CMAKE_MODULE_PATH = ${CMAKE_MODULE_PATH}" )
message( STATUS "${PROJECT_NAME}_DEPENDS = \"${${PROJECT_NAME}_DEPENDS}\"" )
message( STATUS "BUILD_WITH = \"${BUILD_WITH}\"" )
message( STATUS "INSTALL_DOC = ${INSTALL_DOC}" )
message( STATUS "Change a value with: cmake -D<Variable>=<Value>" )
message( STATUS "-------------------------------------------------------------------------------" )
message( STATUS )
