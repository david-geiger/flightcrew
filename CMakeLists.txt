########################################################
#  
#  This is a CMake configuration file.
#  To use it you need CMake which can be 
#  downloaded from here: 
#    http://www.cmake.org/cmake/resources/software.html
#
#########################################################

cmake_minimum_required( VERSION 2.8 ) 

project( FlightCrew )

set( CMAKE_DEBUG_POSTFIX "d" )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib )
set( PACKAGE_DIRECTORY ${PROJECT_BINARY_DIR}/package )

set( FLIGHTCREW_MAJOR_VERSION 0 )
set( FLIGHTCREW_MINOR_VERSION 7 )
set( FLIGHTCREW_REVISION_VERSION 2 )
set( FLIGHTCREW_FULL_VERSION ${FLIGHTCREW_MAJOR_VERSION}.${FLIGHTCREW_MINOR_VERSION}.${FLIGHTCREW_REVISION_VERSION} )
set( FLIGHTCREW_SOVERSION ${FLIGHTCREW_FULL_VERSION} )
set( FLIGHTCREW_LIBVERSION ${FLIGHTCREW_SOVERSION} )

# Check if platform is 64 bit
if( NOT APPLE )
   if( CMAKE_SIZEOF_VOID_P EQUAL 4 )
     set( 64_BIT_PLATFORM 0 )
   else()
     set( 64_BIT_PLATFORM 1 )
   endif()
endif()

# We need this to make sure g++ and the linker
# create exes that can run on 10.9+
set( CMAKE_OSX_DEPLOYMENT_TARGET "10.9" )

# 10.9 is the required minimum OS X version.
# set( CMAKE_OSX_SYSROOT "/Developer/SDKs/MacOSX10.9.sdk" )
# Universal builds for mac
# set( CMAKE_OSX_ARCHITECTURES "i386;x86_64" )

find_package( Boost COMPONENTS date_time filesystem program_options regex system thread system)
find_package( ZLIB )
find_package( PkgConfig )
pkg_check_modules( XERCES xerces-c>=3.1 )

# The parsing order is significant!

if( Boost_FOUND )
    set( BOOST_LIBS ${Boost_DATE_TIME_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_REGEX_LIBRARY} ${Boost_SYSTEM_LIBRARY} ${Boost_THREAD_LIBRARY} )
    set( BOOST_INCLUDE_DIRS ${Boost_INCLUDE_DIRS} )
else()
    add_subdirectory( src/BoostParts )
    set( BOOST_LIBS BoostParts )
    set( BOOST_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src/BoostParts )
endif()
# Set these so zipios doesn't complain.
set( Boost_DATE_TIME_LIBRARY 1 )
set( Boost_FILESYSTEM_LIBRARY 1 )
set( Boost_PROGRAM_OPTIONS_LIBRARY 1 )
set( Boost_REGEX_LIBRARY 1 )
set( Boost_SYSTEM_LIBRARY 1 )
set( Boost_THREAD_LIBRARY 1 )

if( NOT XERCES_FOUND )
    add_subdirectory( src/Xerces )
    set( XERCES_LIBRARIES Xerces )
    set( XERCES_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src/Xerces )
endif()

add_subdirectory( src/XercesExtensions )
set( XERCESEXTENSIONS_LIBRARIES XercesExtensions )
set( XERCESEXTENSIONS_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src/XercesExtensions )

if( NOT ZLIB_FOUND )
    add_subdirectory( src/zlib )
endif()

add_subdirectory( src/zipios )

if( NOT NO_TEST_EXE )
    add_subdirectory( src/googlemock )
endif()

add_subdirectory( src/FlightCrew )
set( FLIGHTCREW_LIBRARIES FlightCrew )
set( FLIGHTCREW_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src/FlightCrew )

add_subdirectory( src/FlightCrew-cli )

if( NOT SKIP_FC_GUI )
    add_subdirectory( src/FlightCrew-gui )
endif()

if( NOT SKIP_FC_PLUGIN )
    add_subdirectory( src/FlightCrew-plugin )
endif()

