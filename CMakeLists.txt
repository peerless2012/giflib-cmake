cmake_minimum_required(VERSION 3.10)
project(giflib VERSION 5.2.2 LANGUAGES C)

option(GIF_BUILD_ENCODER "Also build the GIF encoder (egif_lib.c, gif_hash.c, quantize.c)" OFF)
option(GIF_BUILD_SHARED "Build a shared library instead of a static one" OFF)

# Library only: decoder (+ allocation helpers). The command line tools are not built.
set(GIF_SOURCES
    dgif_lib.c
    gif_err.c
    gifalloc.c
    openbsd-reallocarray.c)
if(GIF_BUILD_ENCODER)
    list(APPEND GIF_SOURCES egif_lib.c gif_hash.c quantize.c)
endif()

if(GIF_BUILD_SHARED)
    add_library(gif SHARED ${GIF_SOURCES})
else()
    add_library(gif STATIC ${GIF_SOURCES})
endif()
set_target_properties(gif PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_include_directories(gif PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>)
target_compile_definitions(gif PRIVATE _GNU_SOURCE)

include(GNUInstallDirs)
install(TARGETS gif
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES gif_lib.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# pkg-config file (name matches what other libs would look up)
set(GIF_PC "${CMAKE_CURRENT_BINARY_DIR}/giflib.pc")
file(WRITE ${GIF_PC}
"prefix=${CMAKE_INSTALL_PREFIX}
libdir=\${prefix}/${CMAKE_INSTALL_LIBDIR}
includedir=\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}

Name: giflib
Description: GIF image library
Version: ${PROJECT_VERSION}
Libs: -L\${libdir} -lgif
Cflags: -I\${includedir}
")
install(FILES ${GIF_PC} DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
