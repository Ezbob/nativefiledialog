cmake_minimum_required(VERSION 3.15)

project(nativefiledialog)

option(USE_ZENITY "Use zenity" OFF)

if (USE_ZENITY)

  # zenity should be cross-platform itself

  set(private_platform_sources
    ${CMAKE_CURRENT_LIST_DIR}/src/nfd_zenity.cpp
  )

else()

  # otherwise we check for platform
  # some assumption:
  # UNIX and GNU compiler -> using GTK
  # UNIX and APPLE -> using cocoa
  # WIN32 -> windows API (COM)

  if (WIN32)
    set(private_platform_sources
      ${CMAKE_CURRENT_LIST_DIR}/src/nfd_win.cpp
    )
  elseif(UNIX)
    if ($<C_COMPILER_ID:GNU>)
      set(private_platform_sources
        ${CMAKE_CURRENT_LIST_DIR}/src/nfd_gtk.cpp
      )
    elseif(APPLE)
      set(private_platform_sources
        ${CMAKE_CURRENT_LIST_DIR}/src/nfd_cocoa.m
      )
    endif()
  endif()

endif()


add_library(nativefiledialog_static STATIC EXCLUDE_FROM_ALL)

target_include_directories(nativefiledialog_static
  PRIVATE ${CMAKE_CURRENT_LIST_DIR}/src/
  PUBLIC ${CMAKE_CURRENT_LIST_DIR}/src/include/
)
target_sources(nativefiledialog_static
  PRIVATE
    ${private_platform_sources}
    ${CMAKE_CURRENT_LIST_DIR}/src/nfd_common.c
)

set_target_properties(nativefiledialog_static
  PROPERTIES
    LIBRARY_OUTPUT_NAME libnfd
)


add_library(nativefiledialog STATIC EXCLUDE_FROM_ALL)

target_include_directories(nativefiledialog
  PRIVATE ${CMAKE_CURRENT_LIST_DIR}/src/
  PUBLIC ${CMAKE_CURRENT_LIST_DIR}/src/include/
)
target_sources(nativefiledialog
  PRIVATE
    ${private_platform_sources}
    ${CMAKE_CURRENT_LIST_DIR}/src/nfd_common.c
)

set_target_properties(nativefiledialog
  PROPERTIES
    LIBRARY_OUTPUT_NAME libnfd
)