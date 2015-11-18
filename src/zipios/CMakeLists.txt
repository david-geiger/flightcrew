########################################################
#  
#  This is a CMake configuration file.
#  To use it you need CMake which can be 
#  downloaded from here: 
#    http://www.cmake.org/cmake/resources/software.html
#
#########################################################

cmake_minimum_required( VERSION 2.8 ) 

project( zipios )

file( GLOB_RECURSE SOURCES *.cpp *.h ) 

# We need to pick up the stdafx.h file
# and the headers for the linked-to libraries
include_directories( ${CMAKE_CURRENT_SOURCE_DIR}
                     ${BOOST_INCLUDE_DIRS}
                     )

if( ZLIB_FOUND )
    include_directories( ${ZLIB_INCLUDE_DIRS} )
else()
    include_directories( ${zlib_SOURCE_DIR} ${zlib_BINARY_DIR} )
endif()

link_directories ( ${PROJECT_BINARY_DIR}/lib )

if( BUILD_SHARED_ZI )
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

target_link_libraries( ${PROJECT_NAME} ${BOOST_LIBS} )
if( ZLIB_FOUND )
    target_link_libraries( ${PROJECT_NAME} ${ZLIB_LIBRARIES} )
else()
    target_link_libraries( ${PROJECT_NAME} zlib )
endif()


#############################################################################

# "Link time code generation" flags for MSVC
# TODO: split into special cmake file
if( MSVC )
    add_definitions( /DUNICODE /D_UNICODE /D_CRT_SECURE_NO_DEPRECATE )
    
    # This warning is present only at the highest warning level (/W4)
    # and is routinely disabled because it complains about valid 
    # constructs like "while (true)"
    add_definitions( /wd4127 )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:wchar_t-" )
    set( CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Oi /GL" ) 
    set_target_properties( ${PROJECT_NAME} PROPERTIES STATIC_LIBRARY_FLAGS "/LTCG" )

# "Print all warnings" flag for GCC
elseif( CMAKE_COMPILER_IS_GNUCXX )
    add_definitions( -Wall )
endif()

if( BUILD_SHARED_ZI AND UNIX AND NOT APPLE )
    install( TARGETS ${PROJECT_NAME}
        LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}
     )
    file( GLOB files "${CMAKE_CURRENT_SOURCE_DIR}/zipios++/*.h" )
    install( FILES ${files} DESTINATION ${INCLUDE_INSTALL_DIR}/${PROJECT_NAME}/zipios++ )
endif()

