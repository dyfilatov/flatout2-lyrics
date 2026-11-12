cmake_minimum_required(VERSION 3.20)
project(fo2lyrics CXX)

if(NOT CMAKE_SIZEOF_VOID_P EQUAL 4)
  message(FATAL_ERROR "FlatOut 2 is a 32-bit game: configure from an x86 developer environment")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

set(FO2_GAME_DIR "" CACHE PATH "FlatOut 2 install folder. When set, the build copies FO2Lyrics.asi into its scripts folder.")

add_library(FO2Lyrics SHARED
  src/fo2vr.manifest
  src/lyrics.cpp
  src/lyrics_dllmain.cpp
  src/util.cpp
)
set_target_properties(FO2Lyrics PROPERTIES SUFFIX ".asi")
target_compile_definitions(FO2Lyrics PRIVATE FO2_LYRICS_STANDALONE=1 WIN32_LEAN_AND_MEAN NOMINMAX _CRT_SECURE_NO_WARNINGS UNICODE _UNICODE)
target_compile_options(FO2Lyrics PRIVATE /W4 /permissive- /utf-8)
target_link_options(FO2Lyrics PRIVATE /SAFESEH:NO)

if(FO2_GAME_DIR)
  add_custom_command(TARGET FO2Lyrics POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "${FO2_GAME_DIR}/scripts"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "$<TARGET_FILE:FO2Lyrics>" "${FO2_GAME_DIR}/scripts/"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "$<TARGET_PDB_FILE:FO2Lyrics>" "${FO2_GAME_DIR}/scripts/"
    COMMENT "Installing FO2Lyrics.asi into ${FO2_GAME_DIR}/scripts"
  )
endif()
