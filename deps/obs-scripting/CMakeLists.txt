cmake_minimum_required(VERSION 3.16...3.25)

legacy_check()

if(NOT ENABLE_SCRIPTING)
  target_disable_feature(obs-scripting "Scripting support")
  return()
endif()

add_library(obs-cstrcache INTERFACE EXCLUDE_FROM_ALL)
add_library(OBS::cstrcache ALIAS obs-cstrcache)

target_sources(obs-cstrcache INTERFACE cstrcache.cpp cstrcache.h)
target_include_directories(obs-cstrcache INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}")

find_package(SWIG 4 REQUIRED)

add_library(obs-scripting SHARED)
add_library(OBS::scripting ALIAS obs-scripting)

include(cmake/lua.cmake)
include(cmake/python.cmake)

if(NOT ENABLE_SCRIPTING_LUA AND NOT ENABLE_SCRIPTING_PYTHON)
  target_disable_feature(obs-scripting "Scripting support")
  return()
else()
  target_enable_feature(obs-scripting "Scripting support")
endif()

target_sources(
  obs-scripting
  PUBLIC obs-scripting.h
  PRIVATE obs-scripting.c obs-scripting-logging.c obs-scripting-callback.h
          $<$<BOOL:${ENABLE_UI}>:obs-scripting-python-frontend.c> $<$<BOOL:${ENABLE_UI}>:obs-scripting-lua-frontend.c>)

target_compile_definitions(obs-scripting PRIVATE SCRIPT_DIR="${OBS_SCRIPT_PLUGIN_PATH}"
                                                 $<$<BOOL:${ENABLE_UI}>:ENABLE_UI>)
target_compile_features(obs-scripting PRIVATE cxx_auto_type)
target_include_directories(obs-scripting PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

target_link_libraries(obs-scripting PRIVATE OBS::libobs OBS::cstrcache $<$<BOOL:${ENABLE_UI}>:OBS::frontend-api>
                                            $<$<PLATFORM_ID:Windows>:OBS::w32-pthreads> $<$<PLATFORM_ID:Darwin>:objc>)

if(OS_WINDOWS)
  configure_file(cmake/windows/obs-module.rc.in obs-scripting.rc)
  target_sources(obs-scripting PRIVATE obs-scripting.rc)
endif()

set_target_properties_obs(obs-scripting PROPERTIES FOLDER scripting PREFIX "")
