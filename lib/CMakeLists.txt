set(LIB_NAME libcurl)
project( libcurl )

configure_file(${CURL_SOURCE_DIR}/include/curl/curlbuild.h.cmake ${CURL_BINARY_DIR}/include/curl/curlbuild.h)
configure_file(curl_config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/curl_config.h)

transform_makefile_inc("Makefile.inc" "${CMAKE_CURRENT_BINARY_DIR}/Makefile.inc.cmake")
include(${CMAKE_CURRENT_BINARY_DIR}/Makefile.inc.cmake)

list( APPEND HHEADERS
  ${CMAKE_CURRENT_BINARY_DIR}/curl_config.h
  ${CURL_BINARY_DIR}/include/curl/curlbuild.h
  )

if(MSVC)
  list(APPEND CSOURCES libcurl.rc)
endif()

# SET(CSOURCES
# #  memdebug.c -not used
# # nwlib.c - Not used
# # strtok.c - specify later
# # strtoofft.c - specify later
# )

# # if we have Kerberos 4, right now this is never on
# #OPTION(CURL_KRB4 "Use Kerberos 4" OFF)
# IF(CURL_KRB4)
# SET(CSOURCES ${CSOURCES}
# krb4.c
# security.c
# )
# ENDIF(CURL_KRB4)

# #OPTION(CURL_MALLOC_DEBUG "Debug mallocs in Curl" OFF)
# MARK_AS_ADVANCED(CURL_MALLOC_DEBUG)
# IF(CURL_MALLOC_DEBUG)
# SET(CSOURCES ${CSOURCES}
# memdebug.c
# )
# ENDIF(CURL_MALLOC_DEBUG)

# # only build compat strtoofft if we need to
# IF(NOT HAVE_STRTOLL AND NOT HAVE__STRTOI64)
# SET(CSOURCES ${CSOURCES}
# strtoofft.c
# )
# ENDIF(NOT HAVE_STRTOLL AND NOT HAVE__STRTOI64)

set( TARGET_LINK ${CURL_LIBS} )
set( TARGET_DEFINES "BUILDING_LIBCURL" )
set( TARGET_INCLUDE_DIRS "" )

if(HIDES_CURL_PRIVATE_SYMBOLS)
  list( APPEND TARGET_DEFINES "CURL_HIDDEN_SYMBOLS" )
endif()

list( APPEND TARGET_INCLUDE_DIRS ${CURL_SOURCE_DIR}/include )
list( APPEND TARGET_INCLUDE_DIRS ${CURL_BINARY_DIR}/include )
list( APPEND TARGET_INCLUDE_DIRS ${libcurl_SOURCE_DIR} )
list( APPEND TARGET_INCLUDE_DIRS ${libcurl_BINARY_DIR} )

if(USE_ARES)
  list( APPEND TARGET_INCLUDE_DIR ${CARES_INCLUDE_DIR} )
endif()

if(CURL_STATICLIB)
  
  # Static lib
  set(CURL_USER_DEFINED_DYNAMIC_OR_STATIC STATIC)
  
else()
  # DLL / so dynamic lib
  set(CURL_USER_DEFINED_DYNAMIC_OR_STATIC SHARED)

endif()

add_library( libcurl ${CURL_USER_DEFINED_DYNAMIC_OR_STATIC} ${HHEADERS} ${CSOURCES} )

target_include_directories( libcurl PRIVATE ${TARGET_INCLUDE_DIRS} )
target_link_libraries( libcurl PRIVATE ${TARGET_LINK} )
target_compile_definitions( libcurl PRIVATE ${TARGET_DEFINES} )

target_include_directories( libcurl INTERFACE $<BUILD_INTERFACE:${TARGET_INCLUDE_DIRS}> )
target_link_libraries( libcurl INTERFACE ${TARGET_LINK} )
target_compile_definitions( libcurl INTERFACE ${TARGET_DEFINES} )
