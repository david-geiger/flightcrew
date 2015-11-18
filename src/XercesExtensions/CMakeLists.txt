########################################################
#  
#  This is a CMake configuration file.
#  To use it you need CMake which can be 
#  downloaded from here: 
#    http://www.cmake.org/cmake/resources/software.html
#
#########################################################

cmake_minimum_required( VERSION 2.8 ) 

project( XercesExtensions )

file( GLOB SOURCES *.cpp *.h ) 

# We need to pick up the stdafx.h file
# and the headers for the linked-to libraries
include_directories( ${CMAKE_CURRENT_SOURCE_DIR}
                     ${XERCES_INCLUDE_DIRS}
                     ${BOOST_INCLUDE_DIRS} )

link_directories ( ${PROJECT_BINARY_DIR}/lib ) 

if( BUILD_SHARED_XE )
    add_library( ${PROJECT_NAME} SHARED ${SOURCES} )
    set_target_properties( ${PROJECT_NAME} PROPERTIES
        VERSION "${FLIGHTCREW_LIBVERSION}"
        SOVERSION "${FLIGHTCREW_SOVERSION}"
        OUTPUT_NAME "${PROJECT_NAME}"
    )
else()
    add_library( ${PROJECT_NAME} ${SOURCES} )
    if( UNIX AND BUILD_SHARED_FC )
        add_definitions(-fPIC)
    endif()
endif()

target_link_libraries( ${PROJECT_NAME} ${XERCES_LIBRARIES} )

#############################################################################

# "Link time code generation" flags for MSVC
# TODO: split into special cmake file
if( MSVC )
    add_definitions( /DUNICODE /D_UNICODE /W4 )
    
    # This warning is present only at the highest warning level (/W4)
    # and is routinely disabled because it complains about valid 
    # constructs like "while (true)"
    add_definitions( /wd4127 )
    
    # The /Zc:wchar_t- flag can't go into add_definitions
    # because the RC compiler picks it up too and it provokes a name clash
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:wchar_t- /MP")
    set( CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Oi /GL" ) 
    set_target_properties( ${PROJECT_NAME} PROPERTIES STATIC_LIBRARY_FLAGS "/LTCG" )

# "Print all warnings" flag for GCC
elseif( CMAKE_COMPILER_IS_GNUCXX )
    add_definitions( -Wall )
endif()

# needed for correct Xerces header inclusion
if( FORCE_BUNDLED_COPIES OR NOT XERCES_FOUND)
    add_definitions( -DXERCES_STATIC_LIBRARY )
endif()

if( BUILD_SHARED_XE AND UNIX AND NOT APPLE )
    install( TARGETS ${PROJECT_NAME}
        LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}
     )
    file( GLOB files "${CMAKE_CURRENT_SOURCE_DIR}/*.h" )
    install( FILES ${files} DESTINATION ${INCLUDE_INSTALL_DIR}/${PROJECT_NAME} )
    install( FILES ${CMAKE_CURRENT_SOURCE_DIR}/FindXercesExtensions.cmake DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}/cmake/${PROJECT_NAME} )
endif()

